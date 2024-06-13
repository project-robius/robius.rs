+++
title = "Building a ChatGTP client using Rust with Makepad"
description = "Learn how to implement a cross-platform application from scratch with Rust and the Makepad framework"
[extra]
author = "Jorge Bejar"
twitter = "jmbejar"
github = "jmbejar"
mastodon = "https://mastodon.social/@jmbejar"
+++

The Rust ecosystem is making significant strides in developing first-class tools for building production-ready applications compatible with major platforms. While there's still work to be done to match the developer experience offered by other tech stacks, we can already explore our options. You'll be impressed by what Rust can accomplish today.

This is the first post in a series that explores how to create cross-platform applications with [Makepad](https://github.com/makepad/makepad), a notable application framework in the Rust community. It is also [one of the key projects under the Robuis initiative](https://project-robius.github.io/book/#key-community-projects). Although it's still evolving and some aspects are being fine-tuned for general production readiness, we can now create impressive applications in just a few steps.

## Create a new Makepad application

Let’s dive into it! First, create a new binary project using `cargo`:

```shell
$> cargo new mychat
```

Now, we want to add `makepad` as a dependency.

```shell
$> cargo add makepad-widgets --git https://github.com/makepad/makepad --branch rik
```

We are now ready to create the necessary elements for an empty application to run. This requires modifying the existing main.rs and adding a few lines.

The `src/main.rs` file should contain the following:

```rust
fn main() {
    mychat::app::app_main()
}
```

And the `src/lib.rs` file:

```rust
pub mod app;
```

The `src/apps.rs` file is the actual entrypoint for Makepad applications. Here is our first version for this file:

```rust,linenos
use makepad_widgets::*;

live_design! {
    import makepad_widgets::base::*;
    import makepad_widgets::theme_desktop_dark::*;

    App = {{App}} {
        ui: <Window> {
            window: {inner_size: vec2(800, 600)},
            pass: {clear_color: #000}
        }
    }
}

app_main!(App);

#[derive(Live, LiveHook)]
pub struct App {
    #[live]
    ui: WidgetRef,
}

impl LiveRegister for App {
    fn live_register(cx: &mut Cx) {
        makepad_widgets::live_design(cx);
    }
}

impl AppMain for App {
    fn handle_event(&mut self, cx: &mut Cx, event: &Event) {
        let scope = &mut Scope::empty();
        self.ui.handle_event(cx, event, scope);
    }
}
```

You can test our new application by executing the cargo run command. For now, you should see an empty window.

## Understanding our application's anatomy

Looking at the code in `src/app.rs`, we can see various sections. One section includes a call to the `live_design!` macro, provided by Makepad. This is where we define the UI components and layout of our application.

Defining a top-level block named `App` is essential. The behavior of this `App` element, which represents the entire application, is determined by the Rust struct `App`. We'll delve into this shortly. Note that our application has only one `Window` widget instance in the `App` definition, representing the "empty window" you see when running the application.

So, how would we go about displaying a "Hello world!" message? It's simply a matter of adding a `Label` widget instance inside the `Window` block.

```rust,linenos,hide_lines=1-2 4-5
use makepad_widgets::*;

live_design! {
    import makepad_widgets::base::*;
    import makepad_widgets::theme_desktop_dark::*;

    App = {{App}} {
        ui: <Window> {
            window: {inner_size: vec2(800, 600)},
            pass: {clear_color: #000}

            // Adding a label displaying some text
            body = {
                <Label> {
                    margin: 40,
                    text: "Hello World!"
                    draw_text: {
                        color: #000,
                    }
                }
            }
        }
    }
}
```

If you modified the application while it was running, you may have noticed that the changes were immediately reflected. This is thanks to Makepad's built-in Live Design feature which automatically detects UI related changes and "hot reloads" the GUI without any recompilation.

All UI elements should be defined within a block named `body`, which is specified in the `Window` widget.

Let's take a deeper look at the Rust code section of the `src/app.rs` file:

```rust,linenos,linenostart=56
app_main!(App);

#[derive(Live, LiveHook)]
pub struct App {
    #[live]
    ui: WidgetRef,
}

impl LiveRegister for App {
    fn live_register(cx: &mut Cx) {
        makepad_widgets::live_design(cx);
    }
}

impl AppMain for App {
    fn handle_event(&mut self, cx: &mut Cx, event: &Event) {
        let scope = &mut Scope::empty();
        self.ui.handle_event(cx, event, scope);
    }
}
```

The Rust code in this file is connected to the application's live UI code via the `app_main` macro (line 56). Makepad then recognizes the `App` struct as the one representing your application, and everything is tied together.

The `LiveRegister` trait must be implemented (lines 64-68) because the framework needs to know the locations of other `live_design` blocks to load them. Currently, we're only including the `live_design` block included in `makepad-widgets`, giving us access to framework-provided widgets like `Window` and `Label`. However, as your project expands, you'll likely define other parts of your application in different files.

Finally, the `AppMain` trait must also be implemented, at least minimally, as we've done (see lines 70-75). For now, we've implemented the `handle_event` function, which describes what happens when the user interacts with the application. The line `self.ui.handle_event(cx, event, scope);` is important as it invokes the `handle_event` handler function in the internal widget instances, such as our label instance.

In makepad, if you implement a `handle_event()` function, you effectively take control over event handling and propagation. Thus, you must explicitly pass events down into each subwidget (or "child" widget) within a widget, if you want each subwidget to be aware of the event and have the ability to handle or respond to it.
This gives you ultimate power over how events propagate throughout different UI widgets/components in the application.

> A crucial aspect to note in Makepad is the existence of a Draw event. This event is triggered when elements on the screen need rendering. If we fail to pass all events to child elements, they will not display because this event will not reach them.

## Building the user interface for our chat

We aim to create a basic version of a ChatGPT client, so let's begin by designing a simple interface. It should include a text input field for the user's prompt, a submit button to send the input to the ChatGPT API, and a list of messages to display the conversation.

For simplicity, we'll implement these features directly in our existing [`app.rs`](http://app.rs/) file. In reality, larger applications would distribute different parts across multiple files. We'll cover how to organize larger applications effectively in future posts.

Let's start by defining a general layout and adding the text input field and submit button.

```bash,linenos,linenostart=2,hide_lines=2-3 7-8
live_design! {
    import makepad_widgets::base::*;
    import makepad_widgets::theme_desktop_dark::*;

    App = {{App}} {
        ui: <Window> {
            window: {inner_size: vec2(800, 600)},
            pass: {clear_color: #000}

            body = {
                height: Fill,
                width: Fill,
                margin: {top: 40, bottom: 40, left: 100, right: 100},

                show_bg: true,
                draw_bg: {
                    color: #330
                }

                flow: Down,
                spacing: 20,

                messages = <View> {
                    height: Fill,
                    width: Fill,
                    margin: 20,
                }

                prompt = <View> {
                    height: Fit,
                    width: Fill,
                    margin: 20,
                    spacing: 10,

                    prompt_input = <TextInput> {
                        height: Fit,
                        width: Fill,
                        padding: 10,
                        empty_message: "Type a message...",
                    }

                    send_button = <Button> {
                        height: Fit,
                        width: Fit,
                        padding: 10,
                        text: "Send",
                    }
                }
            }
        }
    }
}
```

We have replaced all our body block. If you run the application you should see the following:

![Chat interface](/blog/makepad-chatgpt-first-run.png)

We arranged our layout using one of the Makepad's fundamental building blocks: the `View` widget. One view can have a list of children elements to render in our interface. In our case, we added two nested views and configured the parent to have them vertically organized (indicated by `flow: Down` in line 21).

Those two children views are identified as `messages` (line 24) and `prompt` (line 30). Though we're not using those identifiers yet, they will be necessary for reference in the Rust code later. Observe that the first one uses `height: Fill` and the second `height: Fit`. This succinctly conveys that the messages section should take up all available vertical space, with each message taking only the minimum amount of vertical space required to fit the message content in., excluding the area required for the `prompt` view. The `prompt` view's size relies solely on its inner content.

> A common source of issues when working with Makepad is when you have a view sized with `Fit`, but the inner content uses `Fill`. This can cause something to not be displayed at all. When a widget uses `Fill`, it needs to know the parent's size beforehand to calculate its own size. Conversely, when a widget is sized with `Fit`, it needs to calculate the space of its content, which must be calculated without knowing the parent's size.

Here's an alternate way to organize our `live_design` code:

```rust,linenos,linenostart=2,hide_lines=2-3 29-30 33-43
live_design! {
    import makepad_widgets::base::*;
    import makepad_widgets::theme_desktop_dark::*;

    Messages = <View> {
        // Empty for now
    }

    Prompt = <View> {
        spacing: 10,

        <TextInput> {
            height: Fit,
            width: Fill,
            padding: 10,
            empty_message: "Type a message...",
        }

        <Button> {
            height: Fit,
            width: Fit,
            padding: 10,
            text: "Send",
        }
    }

    App = {{App}} {
        ui: <Window> {
            window: {inner_size: vec2(800, 600)},
            pass: {clear_color: #000},

            body = {
                height: Fill,
                width: Fill,
                margin: {top: 40, bottom: 40, left: 100, right: 100},

                show_bg: true,
                draw_bg: {
                    color: #330
                }

                flow: Down,
                spacing: 20,

                messages = <Messages> {
                    height: Fill,
                    width: Fill,
                    margin: 20,
                }
                prompt = <Prompt> {
                    height: Fit,
                    width: Fill,
                    margin: 20,
                }
            }
        }
    }
}
```

This approach makes our `App` easier to read at first glance. These are specific parts of our user interface that have been assigned the aliases `Messages` (lines 6-8) and `Prompt`(10-26) and can be referred to from other points in the DSL. See into the `body` definitions and check how `Messages` (line 46) and `Prompt` (line 51) are instantiated.

We've opted to extract almost everything into these named UI blocks. However, it's worth noting that we still define (or override) the `height`, `width`, and `margin` where we use `Messages` and `Prompt` (see lines 46-48 and 52-54). This is because these values form part of the layout rules we establish in conjunction with the parent view. But, Makepad is highly flexible, allowing you to override as much content as desired to accommodate your needs in various ways.

## Implementing the interaction

Currently, our application is limited to displaying user interface elements. Interaction is restricted to the text input, with no response when you click the `Send` button. Let's enhance this interface.

We need to make some changes to our UI code. We'll add some labels to the `Message` widget to display responses. We'll also add element identifiers, allowing us to reference them from Rust code later. Check the highlighted lines to spot the elements with identifiers.

```rust,linenos,linenostart=2,hide_lines=2-3 64-94 96, hl_lines=13 28 50 57
live_design! {
    import makepad_widgets::base::*;
    import makepad_widgets::theme_desktop_dark::*;

    Messages = <View> {
        height: Fill,
        width: Fill,
        padding: 20,

        flow: Down,
        spacing: 20,

        user_message_bubble = <RoundedView> {
            visible: false,

            height: Fit,
            width: Fill,
            padding: 10,
            draw_bg: {
                color: #222
            }
            user_message = <Label> {
                height: Fit,
                width: Fill,
            }
        }

        model_message_bubble = <RoundedView> {
            visible: false,

            height: Fit,
            width: Fill,
            padding: 10,
            draw_bg: {
                color: #222
            }
            model_message = <Label> {
                height: Fit,
                width: Fill,
            }
        }
    }

    Prompt = <View> {
        height: Fit,
        width: Fill,
        margin: 20,
        spacing: 10,

        message_input = <TextInput> {
            height: Fit,
            width: Fill,
            padding: 10,
            empty_message: "Type a message...",
        }

        send_button = <Button> {
            height: Fit,
            width: Fit,
            padding: 10,
            text: "Send",
        }
    }

    App = {{App}} {
        ui: <Window> {
            window: {inner_size: vec2(800, 600)},
            pass: {clear_color: #000},

            body = {
                height: Fill,
                width: Fill,
                margin: {top: 40, bottom: 40, left: 100, right: 100},

                show_bg: true,
                draw_bg: {
                    color: #330
                }

                flow: Down,
                spacing: 20,

                messages = <Messages> {
                    height: Fill,
                    width: Fill,
                    margin: 20,
                }
                prompt = <Prompt> {
                    height: Fit,
                    width: Fill,
                    margin: 20,
                }
            }
        }

    }
}
```

Note we are using `visible: false` in some views to hide the messages bubbles (lines 15 and 30). We plan to toggle the visibility once we have some messages to display.

Now we can add the Rust logic to implement the “send button”.

```rust,linenos,linenostart=114
impl AppMain for App {
    fn handle_event(&mut self, cx: &mut Cx, event: &Event) {
        // Added this line
        self.match_event(cx, event);

        let scope = &mut Scope::empty();
        self.ui.handle_event(cx, event, scope);
    }
}

impl MatchEvent for App {
    fn handle_actions(&mut self, cx: &mut Cx, actions:&Actions){
        if self.ui.button(id!(send_button)).clicked(&actions) {
            // Capture the text input value
            let user_prompt = self.ui.text_input(id!(message_input)).text();

            // Set the text of the user message label
            self.ui.label(id!(user_message)).set_text(&user_prompt);
            self.ui.view(id!(user_message_bubble)).set_visible(true);

            // Simulate a model response
            let model_response = "Hello, I am a model response!";
            self.ui.label(id!(model_message)).set_text(model_response);
            self.ui.view(id!(model_message_bubble)).set_visible(true);

            self.ui.redraw(cx);
        }
    }
}
```

Let's examine each of these code blocks in detail. First, we added a call to `self.match_event(cx, event)` in line 117, which allows us to use the simpler form of Event matching/handling. This requires us to implement the `MatchEvent` trait for the `App` struct, which we can then use to check for and handle events like a `Button` click.

Next, we implement `handle_actions` to handle the _clicked action_ emitted by the button when the user clicks it (see lines 125-141). The key idea in Makepad is that widgets consume _events_ from other sources and then can emit _actions_ as needed to communicate with other widgets. In this case, the button instance already received and handled a click event and has emitted a related _clicked action_.
We rely on the `clicked(&actions)` function (line 126) to check if the received actions were actually emited by this button instance.

You may notice how we are relying on the identifiers we have in our `live_design` counterpart. Things like `self.ui.text_input(id!(message_input))`, `self.ui.label(id!(user_message))` and `self.ui.view(id!(model_message_bubble))`. This is the Makepad query system for UI elements from Rust code, which is very confortable to use. Just remember you need to use the appropriate function depending on the widget type you’re looking for. In other words, `self.ui.view(id!(message_input))` won’t return anything because the `message_input` id was used for a `TextInput` rather than a `View`.

As a final note, Makepad has a drawing mode which is quite explicit. So, it is our call to indicate to the framework that there were changes in the labels and views instances that needs to be redraw. Hence, we have an invocation to do it: `self.ui.redraw(cx)`, in the line 139. This is a very simple way to “redraw everything” that is not the most efficient way if you have a much more elaborated UI where only a tiny portion has changed, but it is probably a good way to start for now. Nothing stops you to try later to invoke `redraw` in the individual instances of `Label`, `View` and `TextInput` as necessary.

Since we have changed Rust code we are required to recompile our application and run it again to see the changes. Hopefully, you will notice how fast Makepad applications recompile! This is a luxury to have in the Rust ecosystem thanks to the amount of care the Makepad team puts on it.

![First chat interaction](/blog/makepad-chatgpt-first-interaction.png)

It is working! We need to have a model delivering smarter responses now 🙂

## ChatGPT interaction

It’s time to bring real conversation content to our app. We’re going to use the ChatGPT public API, so you will need to generate a key by signing into [platform.openai.com](http://platform.openai.com). Note the number of allowed requests is based on your [current usage tier](https://platform.openai.com/docs/guides/rate-limits/usage-tiers). Using the free tier, you may have to wait a bit while testing because you only get 3 requests per minute. In any case, our implementation will catch error responses and display them so the user always knows what's going on.

Once you have your API key, let’s implement the request to obtain a model response, by following the [official documentation](https://platform.openai.com/docs/api-reference/chat/create).

```rust,linenos,hide_lines=4-124,hl_lines=2 154
use makepad_widgets::*;
use makepad_micro_serde::*;

live_design! {
    import makepad_widgets::base::*;
    import makepad_widgets::theme_desktop_dark::*;

    Messages = <View> {
        height: Fill,
        width: Fill,
        padding: 20,

        flow: Down,
        spacing: 20,

        user_message_bubble = <RoundedView> {
            visible: false,

            height: Fit,
            width: Fill,
            padding: 10,
            draw_bg: {
                color: #222
            }
            user_message = <Label> {
                height: Fit,
                width: Fill,
            }
        }

        model_message_bubble = <RoundedView> {
            visible: false,

            height: Fit,
            width: Fill,
            padding: 10,
            draw_bg: {
                color: #222
            }
            model_message = <Label> {
                height: Fit,
                width: Fill,
            }
        }
    }

    Prompt = <View> {
        height: Fit,
        width: Fill,
        margin: 20,
        spacing: 10,

        message_input = <TextInput> {
            height: Fit,
            width: Fill,
            padding: 10,
            empty_message: "Type a message...",
        }

        send_button = <Button> {
            height: Fit,
            width: Fit,
            padding: 10,
            text: "Send",
        }
    }

    App = {{App}} {
        ui: <Window> {
            window: {inner_size: vec2(800, 600)},
            pass: {clear_color: #000}

            body = {
                height: Fill,
                width: Fill,
                margin: {top: 40, bottom: 40, left: 100, right: 100},

                show_bg: true,
                draw_bg: {
                    color: #330
                }

                flow: Down,
                spacing: 20,

                messages = <Messages> {
                    height: Fill,
                    width: Fill,
                    margin: 20,
                }
                prompt = <Prompt> {
                    height: Fit,
                    width: Fill,
                    margin: 20,
                }
            }
        }
    }
}

app_main!(App);

#[derive(Live, LiveHook)]
pub struct App {
    #[live]
    ui: WidgetRef,
}

impl LiveRegister for App {
    fn live_register(cx: &mut Cx) {
        makepad_widgets::live_design(cx);
    }
}

impl AppMain for App {
    fn handle_event(&mut self, cx: &mut Cx, event: &Event) {
        // Added this line
        self.match_event(cx, event);

        let scope = &mut Scope::empty();
        self.ui.handle_event(cx, event, scope);
    }
}

impl MatchEvent for App {
    fn handle_actions(&mut self, cx: &mut Cx, actions:&Actions){
        if self.ui.button(id!(send_button)).clicked(&actions) {
            // Capture the text input value
            let user_prompt = self.ui.text_input(id!(message_input)).text();

            // Set the text of the user message label
            self.ui.label(id!(user_message)).set_text(&user_prompt);
            self.ui.view(id!(user_message_bubble)).set_visible(true);
            self.ui.redraw(cx);

            // Replacing the hardcoded response with a real one
            send_message_to_chat_gpt(cx, user_prompt);
        }
    }
}

fn send_message_to_chat_gpt(cx: &mut Cx, message: String) {
    let completion_url = "https://api.openai.com/v1/chat/completions".to_string();
    let request_id = live_id!(SendChatMessage);
    let mut request = HttpRequest::new(completion_url, HttpMethod::POST);

    request.set_header(
        "Content-Type".to_string(),
        "application/json".to_string()
    );

    request.set_header(
        "Authorization".to_string(),
        "Bearer <YOUR_ACCESS_KEY>".to_string()
    );

    request.set_json_body(ChatPrompt {
        messages: vec![Message {content: message, role: "user".to_string()}],
        model: "gpt-3.5-turbo".to_string(),
        max_tokens: 100
    });

    cx.http_request(request_id, request);
}

#[derive(SerJson, DeJson)]
struct ChatPrompt {
    pub messages: Vec<Message>,
    pub model: String,
    pub max_tokens: i32
}

#[derive(SerJson, DeJson)]
struct Message {
    pub content: String,
    pub role: String
}

```

> Remember to use your own OpenAI access key in the line 154

With the addition of `send_message_to_chat_gpt()` (lines 142-164), we can send a request to the ChatGPT API server. Note that we’re not yet handling the response so we can focus on the request part. The `cx.http_request` (line 163) is the mechanism in Makepad to issue regular HTTP requests in a non-blocking manner. This ensures that our application UI won’t be blocked while the response is still pending.

The `HttpRequest::set_json_body()` function (line 157) receives a struct representing the JSON format expected by the server. Note that we define the `ChatPrompt` struct (line 167) for this purpose, and then derive `SerJson` and `DeJson` on them to automatically generate efficient JSON parsing logic for them. You can think of those traits as a simpler version of `Serde`, which we are importing in the line 2.

Let’s now receive the responses and update the user interface:

```rust,linenos,linenostart=114,hide_lines=2-15 59-95
impl MatchEvent for App {
    fn handle_actions(&mut self, cx: &mut Cx, actions:&Actions){
        if self.ui.button(id!(send_button)).clicked(&actions) {
            // Capture the text input value
            let user_prompt = self.ui.text_input(id!(message_input)).text();

            // Set the text of the user message label
            self.ui.label(id!(user_message)).set_text(&user_prompt);
            self.ui.view(id!(user_message_bubble)).set_visible(true);
            self.ui.redraw(cx);

            // Replacing the hardcoded response with a real one
            send_message_to_chat_gpt(cx, user_prompt);
        }
    }

    fn handle_network_responses(
        &mut self,
        cx: &mut Cx,
        responses: &NetworkResponsesEvent
    ) {
        let label = self.ui.label(id!(model_message));
        for event in responses {
            match &event.response {
                NetworkResponse::HttpResponse(response) => {
                    match event.request_id {
                        live_id!(SendChatMessage) => {
                            if response.status_code == 200 {
                                let chat_response =
                                    response.get_json_body::<ChatResponse>().unwrap();
                                label.set_text(
                                    &chat_response.choices[0].message.content
                                );
                            } else {
                                label.set_text(&format!(
                                    "Failed to connect with OpenAI: {:?}",
                                    response.get_string_body()
                                ));
                            }

                            self.ui.view(id!(model_message_bubble)).set_visible(true);
                            self.ui.redraw(cx);
                        },
                        _ => (),
                    }
                }
                NetworkResponse::HttpRequestError(error) => {
                    label.set_text(
                        &format!("Failed to connect with OpenAI {:?}", error)
                    );
                    self.ui.view(id!(model_message_bubble)).set_visible(true);
                    self.ui.redraw(cx);
                }
                _ => ()
            }
        }
    }
}

fn send_message_to_chat_gpt(cx: &mut Cx, message: String) {
    let completion_url = "https://api.openai.com/v1/chat/completions".to_string();
    let request_id = live_id!(SendChatMessage);
    let mut request = HttpRequest::new(completion_url, HttpMethod::POST);

    request.set_header(
        "Content-Type".to_string(),
        "application/json".to_string()
    );

    request.set_header(
        "Authorization".to_string(),
        "Bearer <PUT_YOUR_ACCESS_TOKEN_HERE>".to_string()
    );

    request.set_json_body(ChatPrompt {
        messages: vec![Message {content: message, role: "user".to_string()}],
        model: "gpt-3.5-turbo".to_string(),
        max_tokens: 100
    });

    cx.http_request(request_id, request);
}

#[derive(SerJson, DeJson)]
struct ChatPrompt {
    pub messages: Vec<Message>,
    pub model: String,
    pub max_tokens: i32
}

#[derive(SerJson, DeJson)]
struct Message {
    pub content: String,
    pub role: String
}

#[derive(SerJson, DeJson)]
struct ChatResponse {
    pub id: String,
    pub object: String,
    pub created: i32,
    pub model: String,
    pub usage: Usage,
    pub choices: Vec<Choice>,
    pub system_fingerprint: Option<String>,
}

#[derive(SerJson, DeJson)]
pub struct Usage {
    prompt_tokens: i32,
    completion_tokens: i32,
    total_tokens: i32,
}

#[derive(SerJson, DeJson)]
struct Choice {
    message: Message,
    finish_reason: String,
    index: i32,
    logprobs: Option<String>,
}
```

Makepad's `MatchEvent` trait has a `handle_network_responses()` function, and by implementing it (lines 130-171) we now have a way to network-related events. This function is quite straightforward once we define a `ChatResponse` struct (line 211) to represent the JSON response format coming from ChatGPT.

Once we retrieve the chat message from the response, we set the corresponding `Label` instance's text (line 144). We also make sure that the parent view is visible (line 154) and everything gets redrawn (line 155). The parent visibility was hidden because we only want to display the messages bubbles once we have the messages

If everything goes well with the ChatGPT API server, you should see an interaction like the following one:

![ChatGPT response displayed](/blog/makepad-chatgpt-final-result.png)
_This is what ChatGPT knows about Makepad..._ 🙂

## Wrapping up

Let's take a look at what we have achieved in a few steps. We have a basic application with an easy-to-modify look and feel. In just a few dozen lines of code, we made a working chat client that shows real responses from ChatGPT models. Now, we invite you to try different platforms beyond the native desktop platform that we demonstrated in this post. Follow the instructions in the [Makepad README](https://github.com/makepad/makepad/?tab=readme-ov-file#build--run-instructions) to see how to test this application in Android, iOS, and web.

In a future post, we'll cover how to implement a list of messages to make this app really feel like a true interactive chat app.

**Have fun hacking with Makepad and Rust!**
