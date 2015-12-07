## How To Remove CocoaPods From a Project


#### Step 1

The first thing that you will need to do is remove the `Podfile`, `Podfile.lock`, the `Pods` folder, and the generated `workspace`.

#### Step 2

Next, in the .xcodeproj, remove the references to the `Pods.xcconfig` files and the `libPods.a` file. 

#### Step 3

Within the `Build Phases` project tab, delete the `Check Pods Manifest.lock` section (open) and the `Copy Pods Resources section` (bottom). 
