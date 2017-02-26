# SwiftDI

With the release of Swift as an open-source language with a Linux runtime, Swift joins Java and JavaScript as a language that can be used both in a front-end context (iOS apps) as well as on the server. `SwiftDI` is an attempt to demonstrate one way of implementing [*Dependency Inversion*](https://en.wikipedia.org/wiki/Dependency_inversion_principle) (DI) in Swift to achieve the goal of a single high-level policy module containing all business logic that can be deployed in a range of application settings, including an iOS app, a web app, and a RESTful API.

## Installation

### Workstation setup

- Download the latest Xcode from the [Mac App Store](https://itunes.apple.com/us/app/xcode/id497799835?mt=12).
- Launch Xcode and finish the installation steps.
- To build the iOS app, you'll need to install Cocoapods:

    ```bash
    $ gem install cocoapods
    ```

- To run the web/API apps, you'll need to install [Docker for Mac](https://www.docker.com/products/docker).
    
### Project setup

- Clone the project repo and cd into it:

    ```bash
    $ git clone https://github.com/alexbasson/SwiftDI.git
    $ cd SwiftDI
    ```

- Install the Docker container used for this project:

    ```bash
    $ docker run -itv $(pwd):/projects --name projects -w /projects -p 8089:8089 -p 8090:8090 -p 5984:5984 twostraws/server-side-swift /bin/bash
    ```

- `SwiftDI` is merely a container to hold the various projects, which are added as git submodules. Install those projects:

    ```bash
    $ git submodule update --init
    ```

    If you examine the contents of the `SwiftDI` directory, you should now see the following five projects:

      - [`SwiftDIHLP`](https://github.com/alexbasson/SwiftDIHLP): The high level policy module containing all of the reusable business logic.
      - [`SwiftDIApp`](https://github.com/alexbasson/SwiftDIApp): An iOS app using `SwiftDIHLP` for all high level policy.
      - [`SwiftDIWeb`](https://github.com/alexbasson/SwiftDIWeb): A web app, built using the [Kitura](http://www.kitura.io) web framework, using `SwiftDIHLP` for all high level policy.
      - [`SwiftDIAPI`](https://github.com/alexbasson/SwiftDIAPI): A RESTful HTTP API, built using the [Kitura](http://www.kitura.io) web framework, using `SwiftDIHLP` for all high level policy.
      - [`SwiftDIWebRepositories`](https://github.com/alexbasson/SwiftDIWebRepositories): A library containing repository implementations for use in the web-based projects.

### Building and running the projects

See detailed instructions in the respective `README`'s for each project.

# How To Replicate The DI Patterns Demonstrated In `SwiftDI`


## Some background on Swift dependency managers

Implementing dependency-inversion patterns in Swift requires a couple of extra steps relative to other development environments such as Java or Ruby. Maven and Bundler, respectively, make dependency management easy in those two languages, but in Swift, things are more complicated. In part, this is due the existence of competing dependency managers for the Swift ecosystem. In particular, we'll have to use two:

- Swift Package Manager: This is the official package manager for Swift, but it's still very new (under a year old), and as such, it lacks many useful features. For example, it offers incomplete support for local package development. Moreover, iOS apps do not yet support SwiftPM, by which I mean there is no official way to use the SwiftPM to tell an iOS app about its dependencies.
- Cocoapods: This is an unofficial package manager for Objective-C and Swift packages. It ships as a Ruby gem. It supports iOS, and it supports local package development. However, its package-creation tool appears to be broken.

For this project, we'll employ a hybrid approach: We'll make our high-level policy module using the Swift Package Manager, and then we'll retrofit it as a Cocoapod, which we'll then import into our iOS app.


## Project Creation

We'll call this project `SwiftDI`; feel free to use whatever name you want, and replace all mentions of "SwiftDI" with your project name.

### Create the parent directory

Create a new parent directory to hold the various related projects.

```bash
$ mkdir SwiftDI
$ cd SwiftDI
```

### Create the High-Level Policy module

For the High Level Policy (HLP):

1. Create a directory for the high level policy and `cd` into it:

    ```bash
    $ mkdir SwiftDIHLP
    $ cd SwiftDIHLP
    ```

1. Create a Swift package and corresponding Xcode project (this also initializes `SwiftDIHLP` as a git repository):

    ```bash
    $ swift package init
    ```

1. Edit `Package.swift` to add [Quick](https://github.com/Quick/Quick) and [Nimble](https://github.com/Quick/Nimble) as dependencies:

    ```swift
    // Package.swift
    import PackageDescription

    let package = Package(
        name: "SwiftDIHLP",
        dependencies: [
            .Package(url: "https://github.com/Quick/Quick.git", majorVersion: 1),
            .Package(url: "https://github.com/Quick/Nimble.git", majorVersion: 6)
        ]
    )
    ```

1. Build the package; since this is the initial build, it will download the dependencies, so this may take a few minutes:

    ```bash
    $ swift build
    ```

1. Create `SwiftDIHLP.podspec` and edit it to look something like this:

    ```ruby
    # SwiftDIHLP.podspec
    Pod::Spec.new do |s|
      s.name = 'SwiftDIHLP'
      s.version = '0.0.1'
      s.license = '<LICENSE>'
      s.summary = 'SwiftDI high level policy'
      s.homepage = 'https://github.com/<PATH_TO_REPO>'
      s.ios.deployment_target = '8.0'
      s.source = { path: '.' }
      s.authors = { '<AUTHOR_NAME>' => '<AUTHOR_EMAIL>' }
      s.source_files = 'Sources/**/*.swift'
    end
    ```

1. Creating an `.xcodeproj` file is optional. It allows you to edit the module using an IDE such as Xcode or AppCode. To generate an `.xcodeproj` file:

    ```bash
    $ swift package generate-xcodeproj
    ```

That should complete the initial creation of the high-level policy module; move on to creating the app.

### Creating the iOS App

1. From the root of the `SwiftDI` project, create a new Xcode project (`SwiftDIApp`).

    - Use the 'iOS Single View App' template.
    - Make sure the box to include unit tests is **checked**.

1. `cd` into the App's directory and create a `Podfile`:

    ```bash
    $ cd SwiftDI/SwiftDIApp
    $ pod init
    ```

1. Edit the `Podfile` to pull in the `SwiftDIHLP` cocoapod, as well as `Quick` and `Nimble` for testing:

    ```ruby
	platform :ios, '9.0'

	target 'SwiftDIApp' do
	  use_frameworks!

	  pod 'SwiftDIHLP', path: '../SwiftDIHLP'

	  target 'SwiftDIAppTests' do
	    inherit! :search_paths

	    pod 'Quick'
	    pod 'Nimble'
	  end

	end
    ```

1. Install the pods:

    ```bash
    $ pod install
    ```

    You may have to reset some build settings in order to silence warnings from Cocoapods.

1. Open up `SwiftDIApp.xcworkspace`. If you expand the `Pods` project, you'll see a directory for 'Development Pods'; expand this and you'll see the the `SwiftDIHLP` pod.


## Workflow

You'll need to keep two workspaces open—one for the `SwiftDIHLP` module, and one for the `SwiftDIApp` app.

You'll do all the development of the `SwiftDIHLP` module in its workspace. Only its workspace has access to its test suite; the `SwiftDIApp` app will only have access to its source files, not its test suite. As you make changes to the `SwiftDIHLP` module, those changes will automatically appear in the `SwiftDIApp`, so you can just switch back and forth between them as necessary.

## Test-driving the SwiftDIHLP module

Open up `SwiftDIHLP.xcworkspace`. Go to `Tests > SwiftDIHLPTests > SwiftDIHLPTests.swift`. Edit the file to look like this:

```swift
import Quick
import Nimble
@testable import SwiftDIHLP

class SwiftDIHLPTests: QuickSpec {
    override func spec() {
        describe("it works") {
            it("it prints hello world!") {
                expect(helloWorld()).to(equal("Hello, world!"))
            }
        }
    }
}    
```

This should fail to build, but only for the reason that `helloWorld()` doesn't exist yet in `SwiftDIHLP`.

Go to `Sources > SwiftDIHLP > SwiftDIHLP.swift` and edit it to look like this:

```swift
public func helloWorld() -> String {
    return ""
}
```

Type `Cmd-B` to build the project, and return to the test file. Type `Cmd-U` to run the test, and it should fail because the function is returning the empty string. Fix the error, re-run the test, and see it pass.

## Using SwiftDIHLP in SwiftDIApp

Open up `SwiftDIApp.xcworkspace` and go to `ViewController.swift`. Edit the file to look like this:

```swift
import UIKit
import SwiftDIHLP

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        print(helloWorld())
    }

}
```

Type `Cmd-R` to run the app, and once the simulator launches and runs, you should see 'Hello, world!' in the console. Yay!


## Contract Tests

### Setting up contract tests in the HLP module

- You'll need to write your contract tests using `XCTest` rather than `Quick/Nimble` (until and unless we can figure out how to import `Quick/Nimble` into client projects).
- Your test class needs to use the `open` access modifier (not `public`), because we're going to be subclassing it outside its module.
- Include your contract tests (such as tests for repositories) under the Sources directory hierarchy of your high level policy module. E.g. `SwiftDIHLP/Sources/SwiftDIHLP/Contracts/RepositoryTests.swift`
- Add `${PLATFORM_DIR}/Developer/Library/Frameworks` to the HPL module's 'Framework Search Paths' build setting.
- Link the high level policy against `XCTest.framework`.

### Importing contract tests in the client module

- If you're using Cocoapods to import the HLP into the client module:
    - Add `${PLATFORM_DIR}/Developer/Library/Frameworks` to the 'Framework Search Paths' build setting for **both** the client module's target as well as the HLP Pod's target.
    - Link both targets against `XCTest.framework`.


## Running Tests from the Command Line

For the app:

```bash
$ xcodebuild test -workspace SwiftDIApp.xcworkspace -scheme SwiftDIApp -destination 'platform=iOS Simulator,name=iPhone SE'
```

For the high-level policy module:

```bash
$ xcodebuild test -workspace SwiftDIHLP.xcworkspace -scheme SwiftDIHLP -destination 'platform=macOS,arch=x86_64'
```
