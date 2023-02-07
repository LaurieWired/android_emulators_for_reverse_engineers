# Android Emulators for Reverse Engineers

Choosing the right Android emulator for reverse engineering requires considering multiple factors. It's important to ensure that your host machine meets the recommended system requirements, is running a compatible operating system (OS), and can support the architecture of the emulator you plan to use.

**NOTE:** The host machine refers to the machine that will be running the Android emulator, while the guest machine refers to the actual Android emulator running on the host.

## Host Machine Requirements

### Hardware and System Requirements
The following system requirements are recommended for running Android emulators smoothly. Not meeting these minimum requirements may result in severe performance issues.
- 16 GB RAM
- 16 GB disk space
- Virtualization (VT-x) enabled
- Linux (preferable), Windows, or Mac

For optimal performance, it's best to run the emulator on the host machine directly, instead of inside a virtual machine (VM). However, if you must use a VM, most options below require the VM to be Linux for the emulator to successfully boot.

### Operating System
Using a Linux host machine is preferred for performance and compatibility with the Android device. Since Android is based on Linux, most Linux commands will directly translate to Android and vice versa. This may be a pleasant surprise if you are new to Android or Linux analysis. Additionally, some Android analysis tools only run on Linux. If you do still need to use Windows or MacOS, there are many options available for running an emulator on your desired OS.

### Instruction Set Architecture
When deciding which emulator to use, first consider the Instruction Set Architecture (ISA) of the Android application (APK) that you want to analyze. It's important to ensure that the host machine architecture matches the architecture of the emulator you plan to use. Read more about ISAs here: [What Is an Instruction Set Architecture?](https://www.arm.com/glossary/isa). Most real Android devices run ARM, but APKs can support multiple different processor architectures including the following:

- ARM (most popular)
- ARM64
- x86
- x86_64 (least popular)

There are also many different versions of these architectures in between, but these are the main ones to consider.

When selecting from the following emulator software options, **ensure that the host machine architecture matches the architecture of the emulator you plan to use**. For example, running an ARM emulator on an x86_64 host machine will result in poor performance and high resource consumption. The exception to this is between 32-bit and 64-bit architectures since hosts support backward compatibility.

**NOTE:** Due to this requirement, you may need to use multiple different emulator options at any given time, depending on the supported architectures of the APK you are trying to reverse engineer.

### So let's get started!

## Android Device Options

When evaluating the different emulator options, use the following criteria to avoid significant performance issues that make the device unusable:
- x86_64 host: select an x86 or x86_64 emulators
- ARM64 host: select an ARM or ARM64 emulators
- x86 host: select an x86 (32-bit) emulator
- ARM host: select an ARM (32-bit) emulator

### Option 1: Android SDK

The Android SDK (Software Development Kit) is a collection of tools and resources that developers can use to build, test, and debug Android apps. When you install Android Studio on your system, the Android SDK will be installed as well. From inside Android Studio you can create an Android Virtual Device (AVD) with the phone and API of your choice. If you prefer the command line, you can also use the emulator executable to start your device:
```
ANDROID_HOME/sdk/emulator/emulator @AVD_NAME
```
The command above assumes you have already created an AVD. Using the command line is generally preferable as it is less resource-intensive since it does not require running the entirety Android Studio.

Pros
- Plenty of installation and user guides available
- Easy to use Graphical User Interface (GUI)
- Provides emulators for both x86 and ARM-based hosts

Cons
- Many dependencies
- Resource-intensive
- Cannot select an ARM device on x86 host and vice versa

This option is great for beginners new to reverse engineering Android applications since it doesn't require the use of the command line. However, if you need to dynamically run an APK that only supports a different ISA from your host, trying to use an emulator for another architecture will result in poor performance, making it almost impossible to complete your analysis.

### Option 2: Docker
Docker is a containerization platform that allows users to quickly create and destroy containers. It eliminates the need for users to install any dependencies (other than Docker itself) on the host. With Android containers, developers can package all dependencies into a dedicated container and even automate the startup of the Android emulator. Some excellent Android Docker container options include:
- https://github.com/budtmo/docker-android
    - Supports: x86
    - Emulator runs by default without commands
    - Connect to emulator UI through browser
- https://hub.docker.com/r/androidsdk/android-30
    - Supports: x86, x86_64, ARM, ARM64
    - Requires command-line input
    - Needs scrcpy installed on host for emulator UI
- https://github.com/remote-android/redroid-doc
    - Supports: ARM64, x86_64
    - Does not run on Windows even through WSL
    - Needs scrcpy installed on host for emulator UI

The primary advantage of using Docker containers for Android emulators is that you don't have to install any dependencies on the host. This is very helpful if you need to constantly switch between machines. With a new machine, you can simply install Docker and download the Android image of your choice, rather than installing Java, Android Studio, the SDK platform tools, etc. Additionally, you can easily create a custom Dockerfile and automate the creation, running, and installation of APKs on your emulator. To summarize:

Pros
- No dependencies (except Docker itself)
- Easy destruction and recreation of emulator containers
- Automated running of scripts inside Docker containers

Cons
- No user-friendly GUI
- Will not run on Windows except under Windows Subsystem for Linux (WSL)

### Option 3: Android-x86

[Android-x86](https://www.android-x86.org/) is an open source project that releases many different x86-based Android architecture installers. All you need to do is download the ISO disk image and create a new VM in the hypervisor of your choice (example: Hyper-V, VMWare).

Pros
- Only option if host is a Windows VM
- Runs in a hypervisor
- Able to create emulator checkpoints

Cons
- Only supports x86 / x86_64
- Painful install process

The main selling point to this option is that if you for some reason must be running on a Windows VM, you can still run an Android emulator with decent performance so long as the host supports nested virtualization. Additionally, you can use the normal checkpoint options if you want to save the state of your emulator after installing a particular APK for example. The main reason this option is frustrating, however, is that you have to install the entire Android OS at least once. Yes, this means partitioning the drives and doing the whole installation process which will usually take around 30 minutes. After you have completed this once, make sure to create a VM checkpoint to avoid having to repeat the process if something fails.

### Option 4: Cloud Virtual Machine
This option is extremely helpful if you need to run an emulator with an instruction set that does not match your host. For example, if you need to dynamically analyze an APK that will only run on ARM, but you have an x86_64 host, you might think that you are forced to use static analysis. This would produce a long and painful analysis process. Instead, you could spin up a VM inside Amazon Web Services (AWS), Azure, or the cloud provider of your choice with the appropriate ISA.

Pros
- This might be your only option
- Run alternative architectures to your host

Cons
- Costs money

The primary advantage to this option is fairly intuitive. If you must run an APK on an architecture incompatible with your host, you will need a separate machine based on that architecture. Rather than buying another whole computer, pay for only the time you spend analyzing on a cloud VM and delete the VM once your analysis is complete. The best practice would be to only use this option for the specific binaries requiring a different architecture and use one of the other options for all other APKs that will run on your host. Additionally, if using a cloud VM, it is easiest to use Docker to run the emulator so all you need to do is install one type of software on the VM to speed up analysis. The following include details on running ARM64 VMs inside of Azure and AWS:
- [Azure ARM64 Ubuntu](https://ubuntu.com/blog/ubuntu-supports-arm64-based-microsoft-azure-virtual-machines)
- [AWS ARM64 Ubuntu](https://aws.amazon.com/marketplace/pp/prodview-fzujhxzys54w6#pdp-reviews)

### Option 5: Physical Android Device
While you might think this using a real android device would be the best option, there are actually many limitations and potential downsides you encounter with this option:

Pros
- Most accurate device feel
- Bypass anti-emulator checks
- Likely running ARM

Cons
- Phones cost money
- Requires device rooting
- Potentially brick your device
- No way to easily reset device state after running malware
- Limited API and architecture

For most malware analysis, it is preferable to use an emulator since you can select devices that are already rooted and there is no risk of monetary loss if you make a mistake and break the device. Additionally, if you are regularly installing and dynamically analyzing malware, you don't have an easy way to isolate or reset your device once you're finished. This means that you could potentially end up infecting your devices connected to the network or beaconing out to malicious URLs / apps. The main advantage to this option is that you can analyze APKs that only support ARM without performing your analysis on a cloud VM.

## Conclusion
Even though picking an emulator is the first step of dynamic analysis, the amount of possibilities can be overwhelming. Not only that, but you might find yourself in a situation where you are forced to analyze APKs inside of a VM or requiring a different architecture than what is supported by your host machine. The list below provides a summary of the best emulator software to choose for many common situations:
- Running a physical x86 / ARM-based machine
    - Use the Android SDK
- Running an x86-based host but need to analyze an APK that only supports ARM
    - Option 1: use a cloud VM + Docker
    - Option 2: use a physical Android device
- Running inside a Windows VM
    - Use Android-x86
- Need to constantly keep switching analysis machines
    - Use Docker
- Want to create emulator checkpoints
    - Use Android-x86 in a hypervisor
