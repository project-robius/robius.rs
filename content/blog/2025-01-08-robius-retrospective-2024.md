+++
title = "Project Robius in 2024: one year of Rust App Dev"
description = "A retrospective on our progress in the world of Rust App Dev in 2024."
[extra]
author = "Kevin Boos"
twitter = "project_robius"
mastodon = "https://mastodon.social/@kevinaboos"
github = "kevinaboos"
+++

*Author: [Kevin Boos](https://github.com/kevinaboos). Last updated on January 9th, 2024.*

This past year marked the first year of work on **Project Robius**: an open-source decentralized endeavor to enable developers to write immersive, fully-featured apps in pure 🦀&nbsp;Rust&nbsp;🦀 that work seamlessly across all major platforms.

> <font style="bold" color="#C85911"> **ℹ️ Project Robius in a nutshell** </font>
>
> While GUIs are the foundation of an app, there is more to developing a modern featureful app than just drawing a UI.
> Project Robius aims to fill in the gaps in the Rust app dev ecosystem by focusing on everything *except*  the UI, such as abstractions for platform features & OS services, build & packaging tooling, and more.
> We leave UI work to the experts behind the many excellent Rust UI toolkits under active development.

In this post, we'll take a look back on what we've accomplished so far to make the world of Rust App Dev a little better:
1. The crates we've published for accessing platform-provided features from Rust code
2. The major apps we've built using [Makepad] + Robius together
3. The contributions we've made to existing open-source projects in the App Dev space
4. The connections we've fostered throughout the Rust community

We'll also take a deeper look at **Robrix**,  a multi-platform [Matrix](https://matrix.org/) chat client written from scratch in Rust using the [Makepad UI toolkit] and Robius components.

## 1. Robius crates for platform feature abstractions

As our primary objective, we have published several crates intended to be used directly by app devs to access a given platform feature or OS service from their app. The main goal here is for each crate to offer a safe, platform-agnostic abstraction, such that the app dev need not worry about writing any platform-specific code or dealing with each platforms' idiosyncracies.

We began working on these crates in late Spring of 2024.
While development started out gradually, we are significantly ramping up our efforts for the coming year and expect to publish more frequently. That being said, here is the current list:

* [`robius-location`]: access the current geolocation of the user's device
* [`robius-authentication`]: display a native biometric or password authentication prompt
* [`robius-open`]: open a URI or file in a different app (determined by the system)
* [`robius-directories`]: access platform-standard directory locations for app data, user data, config, cache, etc
    * a fork of the [`directories`] crate that adds support for Android
* [`robius-url-handler`]: register your Rust app as the default handler for a URL scheme or file association
* [`robius-keychain`]: store and retrieve secure data/passwords from the platform's secure storage facility
    * This crate was written from scratch, but we discovered [`keyring-rs`] shortly after finishing it. We intend to contribute our additional features, mostly Android support, into `keyring-rs`.
* [`robius-dialog`]: (WIP) display a native dialog for showing a message or allowing the user to pick a file, directory, image, etc.
    * This offers a custom implementation for iOS and Android, but uses [`rfd`] under the hood for Desktop platforms.

We've also released some "lower-level" crates that aren't intended for direct use by an app developer, but they'd be useful for other developers that want to create their own platform abstractions.
The above crates depend on these in various ways.
* [`android-build`]: enables a Rust crate to automatically build Java code for Android targets, as part of a cargo build process via a `build.rs` build script.
    * The Java classfile(s) can then be used by your Rust app, typically via one of the above platform feature abstraction crates.
* [`robius-android-env`]: abstracts access to Android states owned by various UI toolkits.
    * Many Android platform features require passing in the current activity or accessing the JavaVM or JNI environment state.
    * This crate enables us to write other Rust crates that access Android platform features, such that they work seamlessly across many different UI toolkits.
    * Directly supports [`ndk-context`], an existing crate which is compatible with many other UI toolkit components, e.g., [Winit]
        * This enables all of the above platform feature abstraction crates (plus any crate that depends on `robius-android-env`) to work with Winit-based apps on Android.
    * Compared to `ndk-context`, the `robius-android-env` crate offers a "batteries-included" experience that automatically "just works" with supported UI toolkits, such that the app dev doesn't have to add any code to make things work.

## 2. Apps built in 2024 using Makepad + Robius

We (with help from many collaborators) have built both small proof-of-concept demo apps and larger "flagship" apps using Makepad + Robius. Two of the most complex flagship apps we've been developing in 2024 are:
* [Robrix]: a Matrix chat client for power users
* [Moly]: a local LLM chat runtime and AI agent explorer (previously "Moxin")

Both of these apps are fully open-source and have releases available on their GitHub pages linked above, in case you'd like to download and try them out. Note that it's best to build them from source for the most up-to-date experience.


### Robrix: an up-and-coming Matrix chat client for power users

We started [Robrix] about one year ago with the intention of it being a "flagship" Robius app — one that would help drive the development (and priority) of various Robius components and demonstrate their utility.
Since then, our plans for Robrix have expanded beyond it serving as just a demo app or a basic Matrix client; we discuss our longer-term, multi-stage [vision for Robrix in the next post](../robius-roadmap-2025#robrix-roadmap-for-2025-and-beyond).

Robrix has come a long way over the past year, thanks to 750+ commits from 10 contributors!
Since starting from scratch, we have created a functional Matrix chat client with most fundamental features already complete and working well, as shown by our feature status tracker below.

<a href="/blog/robrix_feature_status_tracker.png">
    <img style="width:98%" src="/blog/robrix_feature_status_tracker.png" alt="Robrix's feature status tracker">
</a>

With these features in place, we have began dogfooding Robrix as a daily Matrix client!

While not all main features are complete, Robrix *does* already have some cool features that help both power users and casual users be more productive.
The biggest unique feature of Robrix is an "IDE-like" desktop UI that can display multiple rooms side-by-side in separate tabs, which can be docked and moved around via drag-n-drop actions.
No more wasted horizontal space!

<a href="/blog/robrix_desktop_ui.png">
    <img style="width:98%" src="/blog/robrix_desktop_ui.png" alt="Robrix side-by-side dockable tab UI">
</a>

Another cool feature is that Robrix's UI can automatically transition to different view layouts based on window size. This enables our single codebase to run seamlessly on desktop and mobile platforms, but you can also use any view on any platform if you want.
For example, we frequently enjoy using the mid-size tablet view (below, left) or the narrow mobile view (below, middle) on a smaller laptop screen too, in addition to on our smartphones (below, right).


<div style="gap: 50px;">
<a href="/blog/robrix_midsize_ui.png">
    <img style="width: 43%" alt="Robrix mid-size UI view" src="/blog/robrix_midsize_ui.png" />
</a>
<a href="/blog/robrix_mobile_view_rooms_list.png">
    <img style="width: 27.5%" alt="Robrix narrow mobile UI view of the rooms list" src="/blog/robrix_mobile_view_rooms_list.png" />
</a>
<a href="/blog/robrix_android_view_single_room.png">
    <img style="width: 25.9%" alt="Robrix narrow mobile UI view on Android of a single room" src="/blog/robrix_android_view_single_room.png" />
</a>
</div>


Beyond a sleek UI, Robrix also leverages multiple Robius crates for deep integration with the native platform:
* `robius-open` to open URLs, images, and downloaded files
* `robius-location` to obtain and share the user's current location in a Matrix room
* `robius-url-handler` to register Robrix as a default handler for the `matrix:` URL scheme (and others)
* `robius-directories` to ensure that we store app data and cached content in the platform-canonical directories
* `robius-keychain` to store a user's login session tokens (this is a WIP)
* `robius-packaging-commands` to help easily build app bundles for desktop platforms using cargo-packager
* In the future, we'll allow users to mark individual rooms as "secret", such that they are hidden behind an authentication prompt provided by `robius-authentication`


In addition to a sleek UI and robust platform integration, Robrix achieves high performance and efficiency thanks to its underlying pure-Rust stack and Makepad's emphasis on lightweight, performant code.
Robrix consistently hits the maximum 120 FPS on an older M1 Macbook Pro, remaining smooth and responsive even when scrolling through 10+ rooms displayed side-by-side.
We achieve this while using only around 26-30% of the system RAM that major Electron-based Matrix desktop clients consume to display a single room.
(Note: these are preliminary figures that require deeper benchmarking analysis before drawing conclusions from them.)


Most importantly, thanks to the power of Makepad and Robius, Robrix has zero platform-specific code.
This makes it easy to maintain and develop features/bugfixes quickly, as you don't have to consider the idiosyncracies of each platform.
Thus, we invite you to check out our codebase and contribute any cool features that you'd love to have!


To learn more about Robrix, check out the following resources:
* [Robrix's GitHub repository](https://github.com/project-robius/robrix)
* [A recent conference talk about Robrix](https://www.youtube.com/watch?v=DO5C7aITVyU) ([PDF slides](https://github.com/project-robius/files/blob/main/GOSIM%20China%202024/Robrix%20Talk%20GOSIM%20China%20October%2017%2C%202024.pdf))
* [Robrix's Project Tracker on GitHub](https://github.com/orgs/project-robius/projects/4/)
* [Chat with us about Robrix on Matrix](https://matrix.to/#/#robius-robrix:matrix.org)



### Moly: chat with local LLMs and custom AI agents

[Moly] (f.k.a. *Moxin*) is a pure Rust GUI client for running local Large Language Models (LLMs) and chatting with various AI agents.
You can discover, browse, and download major open-source AI models:

<a href="/blog/moly_discover_screen.png">
    <img style="width:98%" src="/blog/moly_discover_screen.png" alt="Moly's discover LLM screen">
</a>

and then chat with them *locally* without contacting any hosted LLM service.

<a href="/blog/moly_chat_screen.png">
    <img style="width:98%" src="/blog/moly_chat_screen.png" alt="Moly's LLM chat screen">
</a>


Like Robrix, Moly was started about one year ago completely from scratch, and has been a significant driver for the development of fundamental Makepad widgets, components, and Robius infrastructure.
For example, Project Robius contributions to Moly and to other projects (at the request of Moly) were driven by these needs:
* Better packaging logic and build configuration, which became [`robius-packaging-commands`].
  * This cooperates with `cargo-packager` to generate Moly app bundles for all 3 major desktop platforms.
* Portable Rust installation & setup "scripts" that run before the GUI app starts, which became [`moly-runner`].
  * This was needed to install and configure the [WasmEdge WASM runtime], which is how Moly runs LLMs locally.
  * This is also useful for setting up the complex WasmEdge + Moly development environment in just one click.
* Many new Makepad widgets: modals, pop-up notifications, sliding panels, draggable sliders, etc.
* Standardized app behaviors to be more platform-compliant and canonical, e.g., proper use of app data directories.



To learn more about Moly, check out [this blog post](https://dev.to/zhanghandong/moly-an-open-source-llm-client-implemented-in-pure-rust-1hmd) that demonstrates more cool features, screenshots, and examples of what you can do with Moly.
Due to constraints from the underlying WasmEdge runtime, Moly currently runs only on major desktop platforms (Linux, macOS, Windows), but support for iOS and Android is planned.



## 3. Select contributions to other Rust app dev projects
In addition to creating, maintaining, and publishing our own crates for Rust app dev, we also strive to contribute to and improve existing crates that are already prominently used in the ecosystem.

* We began using and making contributions to [`cargo-packager`], a packaging solution for Rust apps on Desktop target platforms created and open-sourced by Crab-Nebula, the folks behind the excellent Tauri ecosystem
    * [Our contributions](https://github.com/crabnebula-dev/cargo-packager/pulls?q=author%3Akevinaboos) were mostly minor bugfixes and improvements to allow the packaging infrastructure to be configured more flexibly
    * As previously mentioned, we published [`robius-packaging-commands`], a companion to `cargo-packager` that makes it easier to build & configure complex apps
        * Automatically calculates the set of dependencies for Debian `.deb` packages
        * Automatically handles Makepad configuration and resource/asset discovery & bundling
    * We intend to add support for other Desktop package formats, namely Flatpack
    * We also plan to contribute support for generating mobile app bundles, namely Android
* We have made [myriad major contributions](https://github.com/makepad/makepad/pulls?q=author%3Akevinaboos) to the Makepad UI toolkit, as Robrix and Moly are two of the most complex/demanding apps built in Makepad
    * Improvements to `PortalList`, a virtual viewport list with infinite scrolling
        * Better API with more introspection into the positional & visibility state of items in the list, its scrolling state, and its item caching behavior
        * Efficient implementations of smooth scrolling animations, e.g., jump to bottom or jump to a given item index
        * Redesign how items are stored and indexed, and how visible items are tracked
    * Rich text formatting for displaying both HTML and Markdown content
        * Including support for most formatting-relevant tags: (un)ordered lists, strikethrough/underline, coloring, indentation, blockquote, code, etc.
        * Special handling of interactive components like HTML links, which must preserve external formatting
    * Multiple new widgets: avatar images with text fallback, abstractions over rich (HTML) text and plaintext, modals, sliding panes, etc
    * Make writing event handlers more ergonomic by avoiding mutable borrows when querying views/widgets
    * Redesign of underlying Android platform layer to allow external crates to access Android system states
    * Enable correct discovery of resource/asset files in macOS/iOS app bundles
    * Many improvements to `cargo-makepad`, a build tool to generate mobile app packages
        * Overhaul code to generate Android APKs
        * Properly install/configure the NDK toolchain on all 3 desktop platforms, plus enable building native code (via `cc-rs`)
        * Ensure backwards compatibility with standard Android Studio-managed SDKs
    * An improved app lifecycle model with dedicated events for all lifecycle stages, which is consistent across all platforms
    * Easier and more ergonomic `Actions` (widget-to-widget message events)
        * Plus support for delivering an action to a widget from a background thread or async task context
* We made [minor contributions](https://github.com/kornelski/rust-security-framework/pull/210) to the [`security-framework`] crate, which offers Rust bindings to Apple's security framework (for TLS, keychain, etc)
    * We added a few missing APIs to enabling updating or deleting keychain items, which we needed to fully implement [`robius-keychain`]
* We implemented a Rust auto-installer and configurer for the [WasmEdge WASM runtime], as mentioned [above](#moly-chat-with-local-llms-and-custom-ai-agents)
    * This massively simplifies both the developer-side build process and the user installation procedure for Moly, which relies on WasmEdge to run LLMs locally.
    * We hope to transform this into the official install script for WasmEdge and upstream it for general usage there, as much of the effort involved was devoted to extracting the precise system configuration required to select and install the proper WasmEdge release.


## 4. Cross-collaboration with other UI and App Dev orgs
Beyond publishing crates and developing apps, we also want to bring together people of all stripes across the Ruist UI and App Dev ecosystem.
To that end, Project Robius hosted an [App Dev unconference](https://2024.rustnl.org/unconf/) at RustNL 2024 (and also GOSIM Beijing 2024), in which a few dozen Rust developers from across the world met up to discuss the shared problems we all face in developing Rust apps and UI toolkits.
We discussed everything from build tooling to text layout, accessibility, Winit compatibility, and more.
A few of the topics & ideas from the unconference(s) have already made it past the discussion phase and have become real projects!
* [`kittest`]: a universal UI testing framework built upon the [AccessKit] accessibility framework, spearheaded by the eGUI team!
* [Dioxus's work on hotreloading](https://www.reddit.com/r/rust/comments/1ezdjqx/media_i_added_instant_hotreloading_of_some_rust/) not just UI DSL code, but even real Rust code that implements app behavior!
* Feedback given to the Rust project teams, primarily lang, libs, and compiler.
    * We focused on changes to Rust that will make future Rust apps easier to write, with simplified and more ergonomic code patterns for async and more.

In addition, thanks to our colleague Sid Askary, we began monthly meet-ups to chat about ongoing Rust UI & App Dev concerns, and to share ideas, solutions, progress updates.
Attendees vary, but often include teammembers from [Robius](https://robius.rs/), [Makepad](https://makepad.nl/), the [Linebender organization](https://linebender.org/) (behind Xilem and more), [Dioxus](https://dioxuslabs.com/), [eGUI](https://github.com/emilk/egui), [Pax](https://www.pax.dev/), [wgpu](https://github.com/gfx-rs/wgpu), [Slint](https://slint.dev/), and more.
If you're in the Rust App Dev or UI space and would like to join future meetups, consider getting in touch!



## Our Roadmaps for 2025

[Check out our next blog post](../robius-roadmap-2025) for roadmaps for both Project Robius and Robrix in 2025 (and beyond).


## Acknowledgments
If you made it this far, thanks for reading! You must be a true fan of Rust app dev 😊!

Before we depart, I'd like to thank the following key people who have been instrumental to the success of Project Robius over the past year.

* The Makepad team: [Rik Arends](https://x.com/rikarends), [Eddy Bruël](https://github.com/ejpbruel2), [Sebastian Michailidis](https://twitter.com/SebMichailidis)
* [Klim Tsoutsman](https://github.com/tsoutsman)
* [WyeWorks](https://www.wyeworks.com/) developers: [Jorge Bejar](https://github.com/jmbejar),  [Julián Montes de Oca](https://github.com/joulei), [Facundo Mendizábal](https://github.com/fmzbl)
* [Alex Zhang (ZhangHanDong)](https://github.com/ZhangHanDong) and his team members: [@alanpoon](https://github.com/alanpoon), [@aaravlu](https://github.com/alanpoon), [@tyreseluo](https://github.com/tyreseluo), [@Guocork](https://github.com/Guocork)
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