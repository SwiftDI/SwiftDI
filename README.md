# SwiftDI

## Some background on Swift dependency managers

Implementing dependency-inversion patterns in Swift requires a couple of extra steps relative to other development environments such as Java or Ruby. Maven and Bundler, respectively, make dependency management easy in those two languages, but in Swift, things are more complicated. In part, this is due the existence of competing dependency managers for the Swift ecosystem. In particular, we'll have to use two:

- Swift Package Manager: This is the official package manager for Swift, but it's still very new (under a year old), and as such, it lacks many useful features. For example, it offers incomplete support for local package development. Moreover, iOS apps do not yet support the SPM, by which I mean there is no official way to use the Swift Package Manager to tell an iOS app about its dependencies.
- Cocoapods: This is an unofficial package manager for Objective-C and Swift packages. It ships as a Ruby gem. It supports iOS, and it supports local package development. However, its package-creation tool appears to be broken.

For this project, we'll employ a hybrid approach: We'll make our high-level policy module using the Swift Package Manager, and then we'll retrofit it as a Cocoapod, which we'll then import into our iOS app.

## Workstation setup

- Download the latest Xcode from the [Mac App Store](https://itunes.apple.com/us/app/xcode/id497799835?mt=12).
- Launch Xcode and finish the installation steps.
- Install Cocoapods:

    ```bash
    $ gem install cocoapods
    ```
    

## Project Creation

We'll call this project `SwiftDI`; feel free to use whatever name you want, and replace all mentions of "SwiftDI" with your project name.

1. Create a new parent directory to hold the `.xcodeproj` projects and the `.xcworkspace`s that will contain them.

    ```bash
    $ mkdir SwiftDI
    ```

1. Initialize a git repository in the parent directory.

    ```bash
    $ cd SwiftDI
    $ git init
    ```

1. Add the Swift `.gitignore` [from Github](https://github.com/github/gitignore/raw/master/Swift.gitignore)
    
1. For the High Level Policy (HLP):

    1. Create a directory for the high level policy and `cd` into it:

        ```bash
        $ mkdir SwiftDIHLP
        $ cd SwiftDIHLP        
        ```
        
    1. Create a Swift package and corresponding Xcode project:

        ```bash
        $ swift package init
        $ swift package generate-xcodeproj
        ```
        
    1. Initialize a Podfile:

        ```bash
        $ pod init
        ```
        
    1. Edit `Podfile` to pull `Quick` and `Nimble` into the HLP module:

        ```ruby
        target 'SwiftDIHLP' do
          use_frameworks!

          target 'SwiftDIHLPTests' do
            inherit! :search_paths

            pod 'Quick'
            pod 'Nimble'
          end

        end        
        ```
        
        Install the pods:
        
        ```
        $ pod install
        ```
        
        **N.B.** You may get some warnings regarding the `LD_RUNPATH_SEARCH_PATHS`. To fix this, open up `SwiftDIHLP.xcworkspace` in Xcode, and for each of the two targets under the `SwiftDIHLP` project (namely `SwiftDIHLP` and `SwiftDIHLPTests`), do the following:
        
        - Click on "Build Settings"
        - Search for "LD\_RUNPATH\_SEARCH\_PATHS"
        - Double-click on the value for the setting
        - Add "$(inherited)" to the existing values

        Once you've done this for each of the targets, run `$ pod install` again; you should see no warnings.
        
    1. Create `SwiftDIHLP.podspec` and edit it to look something like this:

        ```ruby
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
        
        That should complete the initial creation of the high-level policy module; move on to creating the app.        

1. For the iOS App, create a new Xcode project (`SwiftDIApp`).

    - Use the 'iOS Single View App' template.
    - Make sure the box to include unit tests is **checked**.
    - Make sure the box to create a new git repository is **unchecked**.

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
    
    Again, you may have to reset some build settings in order to silence warnings from Cocoapods.
    
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

### setting up contract tests in the HLP module

- You'll need to write your contract tests using `XCTest` rather than `Quick/Nimble` (until and unless we can figure out how to import `Quick/Nimble` into client projects.
- Your test class needs to use the `open` access modifier (not `public`), because we're going to be subclassing it outside its module.
- Include your contract tests (such as tests for repositories) under the Sources directory hierarchy of your high level policy module. E.g. `SwiftDIHLP/Sources/SwiftDIHLP/Contracts/RepositoryTests.swift`
- Add `${PLATFORM_DIR}/Developer/Library/Frameworks` to the HPL module's 'Framework Search Paths' build setting.
- Link the high level policy against `XCTest.framework`.

### importing contract tests in the client module

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
