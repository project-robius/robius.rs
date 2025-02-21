+++
title = "Our Roadmaps for Project Robius & Robrix in 2025"
description = "A look ahead at our future vision, challenges, and goals for Project Robius and Robrix."
[extra]
author = "Kevin Boos"
twitter = "project_robius"
mastodon = "https://mastodon.social/@kevinaboos"
github = "kevinaboos"
+++

*Author: [Kevin Boos](https://github.com/kevinaboos). Last updated January 16th, 2024.* [Discuss this on Reddit](https://www.reddit.com/r/rust/comments/1i2wq5j/project_robius_in_2024_another_year_of_progress/).

‼️ **This is a continuation of [our previous post](../robius-retrospective-2024)**.

We've just completed the first full year of work on **Project Robius**: an open-source decentralized endeavor to enable developers to write immersive, fully-featured apps in pure 🦀&nbsp;Rust&nbsp;🦀 that work seamlessly across all major platforms.

In this companion post, we'll take a look at our roadmap for 2025 and beyond, both for Project Robius as a whole and for the [Robrix] app specifically.


## Project Robius Roadmap for 2025

Project Robius in 2025 aims to continue the work we've begun in 2024 to improve the overall app dev experience in Rust.

#### More Rust abstractions for platform features
As a technical organization, our primary ongoing focus will be to keep creating and publishing as many high-quality platform feature abstraction crates as possible.
Now that we have a foundation established with [several existing crates](../robius-retrospective-2024#1-robius-crates-for-platform-feature-abstractions), we anticipate being able to make faster progress, especially with the expected addition of more contributors.
Our targeted platform features include (in rough priority order):
* File/image/media picker (in progress)
* Native system notifications (in progress)
* Toasts, pop-up messages, status bar icons
    * We have implemented this in Makepad, but not with native widgets commonly used on Mobile platforms
* Spawning long-running background tasks/services
* System file/media store
* Native context menus
    * Same status as toasts above.
* Camera access & configuration
* Audio input (microphone)
* System theming choices (e.g., dark mode, key colors)
* Connectivity manager/subscriber
* Power/battery status
* Haptics/vibration


#### Better, more automated build tooling
Another topic dear to our hearts is build tooling.
We aim to improve the state of build tools such that the app developer themself can be relieved from the burden of managing and figuring out platform-specific details, such as which permissions/entitlements their app requires to build and run properly on mobile platforms.
Ideally, we'd like to be able to auto-generate a fully-formed Android XML manifest or Apple `Info.plist` file with all of the necessary permissions that an app requires, without requiring the app dev to possess expert knowledge about the requirements of their app's dependencies and transitive dependencies.

One such idea for realizing this is to make each `robius-*` platform feature abstraction crate automatically emit its required permissions during the build process.
Exactly *how* to export and encode this information is still up in the air, but we have discussed leveraging a linker-based approach similar to what [Dioxus's manganis project](https://github.com/DioxusLabs/manganis) does to encode resource/asset paths into special linker sections.
This would allow a top-level tool to run after the `cargo build` process, and inspect the binary's special linker sections in order to automatically generate a full permissions/entitlements file for the given target platform.
We envision that this could also be used for other arbitrary UI toolkits, not just Makepad, as well as emitted by other platform abstraction crates outside of the `robius-*` organization.



#### Effortless integration with other UI toolkits
In addition, we wish to explore deeper integration and first-class compatibility (and testing pipelines) with other Rust UI toolkits, e.g., Dioxus, eGUI, and more.
Our first year of development has been centered on Makepad, in the sense that we've built two full-size Makepad apps, contributed significantly to Makepad itself, and have focused on test-driving our crates using Makepad apps (see [`robius-demo-simple`]).
Thus, using Robius components in a Makepad app is quite easy for the app developer.
All they need to do is add a dependency on a special "marker" crate that auto-configures all `robius-*` crates to work with Makepad, as shown below in Robrix's `Cargo.toml`:
```toml
[dependencies]
makepad-widgets = { git = "https://github.com/makepad/makepad", branch = "rik" }
robius-open = "0.1.2"
robius-directories = "5.0.1"
robius-location = { git = "https://github.com/project-robius/robius-location" }

## Including this crate automatically configures all `robius-*` crates to work with Makepad.
robius-use-makepad = "0.1.1"
```


Now that we have successfully realized several platform feature abstraction crates, we would like to ensure that these can be easily utilized by apps built in other UI toolkits.
For example, one specific secondary goal for this year is to explore how `robius-*` crates could comprise Dioxus's [`dioxus-std`] library and fill in the gaps in their mobile platform support.


Another related goal is to design a UI-focused concurrency management library with an interface that helps app devs easily write high-performance apps that never block or bog down the main UI thread with long-running operations.
We envision easy interfaces to offload code to background threads or async tasks, as well as for exchanging data between these background contexts and the performance-sensitive the UI main thread.
The inability to easily invoke async functions from the UI main thread (without causing performance hiccups) is a long-running frustration we have had when developing Robrix, as many SDKs are written with a hard dependency on an async executor, typically tokio.
This serves as strong motivation to ameliorate the overly-complex code patterns shown in the diagram below, in which Robrix's structure of multiple execution contexts with myriad distinct communication mechanisms between them must be *manually* managed.

<a href="/blog/robrix_concurrency_diagram.png">
    <img style="width:98%" src="/blog/robrix_concurrency_diagram.png" alt="Diagram of how Robrix must manually manage mixed concurrency contexts">
</a>

> To understand this concurrency challenge in more detail, [watch this presentation on Robrix (starting from 23:10)](https://youtu.be/DO5C7aITVyU?si=N_10UZBCR5g-w2D4&t=1390) or [check out slides 26-33 here](https://github.com/project-robius/files/blob/main/GOSIM%20China%202024/Robrix%20Talk%20GOSIM%20China%20October%2017%2C%202024.pdf).


A key component of this library is an abstract *compile-time token* that statically ensures whether code is executing within the context of the main UI thread context.
Such a type must be both non-`Send` and non-`Sync`, and only possible to construct on the main UI thread.
This token is necessary because most platforms require many of their platform-provided APIs to be invoked on the main thread, and it's significantly better to check this at compile time than via a runtime assertion.
We have realized this for Makepad via a mutable reference to the [context type](https://github.com/makepad/makepad/blob/0084948c176a99740af92a71578543c3fcc0b63f/platform/src/cx.rs#L55) `&mut Cx`, which is created only on the main UI thread and then passed as a mutable reference to all of the [event handlers and draw routines](https://github.com/makepad/makepad/blob/0084948c176a99740af92a71578543c3fcc0b63f/widgets/src/widget.rs#L45-L110).

Here's a simplified example of how we leverage this technique in Robrix to implement an efficient cache for user profile information, while avoiding the need to acquire any locks on the main UI thread.
```rust
/// Returns the cached user profile for the given user ID ...
///
/// This function requires passing in a reference to `Cx`,
/// which isn't used, but acts as a guarantee that this function
/// must only be called by the main UI thread.
pub fn get_user_profile(
    _cx: &mut Cx,
    user_id: OwnedUserId,
    fetch_if_missing: bool,
) -> Option<UserProfile> {
    // access the TLS cache, defined below.
    ...
}

thread_local! {
    static USER_PROFILE_CACHE: RefCell<BTreeMap<OwnedUserId, UserProfileCacheEntry>> = const { ... };
}
```

Robrix also uses this to statically ensure that a location initialization function can only be invoked from the main UI thread:
```rust
/// Starts listening for location requests and updates to the latest device location.
/// ...
pub fn init_location_subscriber(_cx: &mut Cx) -> Result<(), robius_location::Error> {
    ...
}
```
With a proper UI-agnostic abstraction for a statically-known thread context marker, we can do this not just in the Makepad app logic, but also within the `robius-*` platform feature crates themselves.
Finally, while this sort of concurrency library and thread context abstractions are highly desirable, it's also admittedly a longer-term goal that merits major effort beyond just 2025.


#### Organizing more conferences & meet-ups
On the organizational side, we intend to sponsor two more conferences for open-source Rust development and host informal Rust app dev unconferences co-located with those conferences.
These will be scheduled similarly to [the ones we hosted in 2024](../robius-retrospective-2024#4-cross-collaboration-with-other-ui-and-app-dev-orgs):
the first will be [RustWeek 2025](https://rustweek.org/) (formerly "RustNL") in the Netherlands in May, and the second will be [GOSIM China](https://china2024.gosim.org/) in autumn 2025.
With these (un)conferences, we aim to bring community members together again to collaborate, share ideas, and to further advance the state-of-the-art for App Dev and UI in Rust.


<hr style="border: none; width: 100%; color: #000000; background-color: #000000; height: 1px;" >


## Robrix Roadmap for 2025 and beyond

As discussed in [our previous post](../robius-retrospective-2024#robrix-an-up-and-coming-matrix-chat-client-for-power-users), Robrix is an up-and-coming Matrix chat client for power users, and serves as a "flagship" Robius app to drive the development priorities of various Robius components.


While Robrix iisoff to a strong start, we still have a long way to go, and we have a lot more cool features in mind beyond just Matrix chat support.
We have planned several high-level phases of Robrix development over the next 18-24 months:
1. <font color="gray">*[Q1 2025]*</font>&nbsp; Release an alpha version of Robrix with most fundamental Matrix features available.
    * Realize sufficient functionality to be usable as a daily driver, but not yet to be a complete replacement for existing clients.
    * This is nearly complete! See [Milestone 1](https://github.com/project-robius/robrix/milestone/1) on our GitHub page.
2. <font color="gray">*[Summer 2025]*</font>&nbsp; Publish Robrix v1.0 with full Matrix functionality, for "power" users.
    * Offer a responsive UI design with a dockable, multi-tab view of many rooms side-by-side, which also adapts to varying screen sizes (mobile, desktop, etc).
        * ✅ This is already complete! ([as described in our previous post](../robius-retrospective-2024#robrix-an-up-and-coming-matrix-chat-client-for-power-users))
    * Achieve feature parity with existing major clients, including administrative features like a full settings pane, session management, room creation/admin, message search, threads, spaces, etc.
        * See [Milestone 2](https://github.com/project-robius/robrix/milestone/2) on our GitHub page for more details.
        * Generally, these features are *not* drivers of Robius development, as they don't require complex platform features, so they were of a lower priority initially.
    * Distribute Robrix app bundles to platform app stores and package managers.
3. <font color="gray">*[Q3 2025]*</font>&nbsp; Integrate local LLM runtimes (like [Moly](../robius-retrospective-2024#moly-chat-with-local-llms-and-custom-ai-agents)) for powerful, advanced convenience features.
    * LLMs or AI agents can summarize conversations, analyze important topics, and extract key action items from "what you missed" after a holiday. Here's a UI prototype: <br>
        <a href="/blog/robrix_moly_prototype.png">
            <img style="width: 50%" alt="A prototype UI design for AI LLMs alongside Matrix rooms in Robrix" src="/blog/robrix_moly_prototype.png" />
        </a>
    * AI chatbots can assist newcomers in large open-source projects by auto-answering FAQs, either privately or publicly to allow for additional interaction from real expert users.
    * Key point: *fully-local* LLM runtimes **cannot jeopardize end-to-end encrypted (E2EE) rooms or user data sovereignty**, so you can utilize LLMs with confidence that your privacy is being honored.
4. <font color="gray">*[Late 2025]*</font>&nbsp; Go beyond Matrix: Robrix as a central "hub" for federated & open-source services
    * Collect multiple services into a unified app view, including ActivityPub-based microblogs (e.g., [Mastodon]), views of source code and related issues/pull requests, discussion forums (e.g., [Lemmy]), and more.
        * The exact set of supported services are TBD.
    * The availability of many services in a single app context can enable unique combo features, such as a combined activity feed of notifications + news from various sources, or easy one-click broadcasting of project updates to multiple communities across different services.
5. <font color="gray">*[Long-term]*</font>&nbsp; Explore how to use decentralized identity providers like [OpenWallet] to login to Robrix-supported services.
    * Use Robrix as the first experimental testing ground for integrating a device-local wallet app as an ID provider for Matrix authentication.
    * For more info, check out [this presentation by Wenjing Chu, an OpenWallet expert](https://www.youtube.com/watch?v=eq9pnYB5-Xk) from the Matrix Conference 2024.


While many of these are larger endeavors, we anticipate being able to complete at least milestones 1, 2, and 3 by the end of this coming year.


## Acknowledgments
I'd like to thank the following key people who have been instrumental to the success of Project Robius over the past year, and who will undoubtedly help it flourish in 2025.

* The Makepad team: [Rik Arends](https://x.com/rikarends), [Eddy Bruël](https://github.com/ejpbruel2), [Sebastian Michailidis](https://twitter.com/SebMichailidis)
* [Klim Tsoutsman](https://github.com/tsoutsman)
* [WyeWorks](https://www.wyeworks.com/) developers: [Jorge Bejar](https://github.com/jmbejar),  [Julián Montes de Oca](https://github.com/joulei), [Facundo Mendizábal](https://github.com/fmzbl)
* [Alex Zhang (ZhangHanDong)](https://github.com/ZhangHanDong) and his team members: [@alanpoon](https://github.com/alanpoon), [@aaravlu](https://github.com/aaravlu), [@tyreseluo](https://github.com/tyreseluo), [@Guocork](https://github.com/Guocork)
* [Cassaundra](https://github.com/cassaundra)
* My colleagues who provide invaluable guidance, technical advice, and community connections: Yue Chen, Edward Tan, Sid Askary, Yong He, Mats Lundgren
* Linebender teammembers, for technical recommendations and serving as a sounding board for exchanging ideas
* [@smarizvi110](https://github.com/smarizvi110) and other miscellaneous contributors from the open-source community

<hr style="border: none; width: 100%; color: #000000; background-color: #000000; height: 1px;" >

<!-- Links -->
[Robrix]: https://github.com/project-robius/robrix
[Moly]: https://github.com/moxin-org/moly
[Makepad]: https://makepad.nl/
[Makepad UI toolkit]: https://makepad.nl/
[WasmEdge WASM Runtime]: https://wasmedge.org/
[OpenWallet]: https://openwallet.foundation/
[Mastodon]: https://joinmastodon.org/
[Lemmy]: https://join-lemmy.org/
[`moly-runner`]: https://github.com/moxin-org/moly/blob/a82d297b155fa64efd2cdb5d6b14c89148a1c70b/moly-runner/src/main.rs
[`robius-location`]: https://github.com/project-robius/robius-location
[`robius-authentication`]: https://crates.io/crates/robius-authentication
[`robius-open`]: https://crates.io/crates/robius-open
[`robius-directories`]: https://crates.io/crates/robius-directories
[`directories`]: https://crates.io/crates/directories
[`robius-url-handler`]: https://github.com/project-robius/robius-url-handler
[`robius-keychain`]: https://github.com/project-robius/robius-keychain
[`keyring-rs`]: https://crates.io/crates/keyring
[`android-build`]: https://crates.io/crates/android-build
[`robius-android-env`]: https://crates.io/crates/robius-android-env
[`ndk-context`]: https://crates.io/crates/ndk-context
[`Winit`]: https://crates.io/crates/winit
[`robius-packaging-commands`]: https://github.com/project-robius/robius-packaging-commands
[`robius-dialog`]: https://github.com/project-robius/robius-file-dialog
[`rfd`]: https://crates.io/crates/rfd
[`cargo-packager`]: https://crates.io/crates/cargo-packager
[`security-framework`]: https://crates.io/crates/security-framework
[`kittest`]: https://crates.io/crates/kittest
[`robius-demo-simple`]: https://github.com/project-robius/robius-demo-simple
[`dioxus-std`]: https://dioxuslabs.com/learn/0.5/contributing/roadmap/#mobile
[AccessKit]: https://accesskit.dev/