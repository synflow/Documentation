The Synflow IDE (formerly called ngDesign) is the official Integrated Development Environment (IDE) for FPGA IP cores and system development, based on the Cx language and the [Eclipse](https://eclipse.org/downloads/) framework. On top of Eclipse powerful code editor and developer tools, Synflow IDE offers even more features that enhance your productivity when building Ip cores, applications, and systems for FPGAs, such as:

- A flexible, automatic compiling system
- A fast and feature-rich development platform
- An unified environment where you can develop for all FPGAs
- Instant Run to compile and synthesis for any FPGA
- Code templates and Git integration to help you build systems and import existing code
- (Beta) Integrated simulation tool, up to 10 times faster than Verilog simulators
- Instant Run to compile and simulate for [MG Modelsim](https://www.mentor.com/products/fv/modelsim/)

This page provides an introduction to basic Synflow IDE features. For a summary of the latest changes, see [Synflow IDE Release Notes](https://www.synflow.com/changelog).

## Project Structure

Each project in Synflow IDE contains one Synflow Project, and one or more package(s) and folder(s) with source code files and resource files. Types of packages include:

- Hardware modules
- Library modules
- Networks of modules
- Existing (HDL) code

By default, Synflow IDE displays your project files in the Project Explorer view, as shown in figure 1. This view is organized by packages and folders to provide quick access to your project's key source files.

The build files are visible at the top level under specific folders and each Synflow project contains the following folders:

- **src**: Contains the source Cx files and libraries.
- **projects (generated)**: Contains the projects ressources for FPGA Synthesizers.
- **sim (generated)**: Contains the projects ressources for the Synflow simulator (and for MG Modelsim).
- **testbench (generated)**: Contains the projects ressources for MG Modelsim.
- **verilog-gen (generated)**: Contains generated Verilog source files.
- **VHDL-gen (generated)**:  Contains generated VHDL source files.

![The project files in Project Explorer.](/images/synflow/ProjectFiles.png "The project files in Project Explorer.")

You can also customize the view of the project files to focus on specific aspects of your app development. For example, selecting the **Network** view of your project displays how the various tasks of a selected network are connected together.

![The network view.](/images/synflow/NetworkView.png "The network view.")

For more information, see [Managing Projects](https://developer.android.com/tools/projects/index.html).

## User Interface

The Synflow IDE main window is made up of several logical areas identified in figure 3.

![The Synflow IDE main view.](/images/synflow/SynflowIDE.png "The Synflow IDE main view.")

1. The toolbar lets you carry out a wide range of actions, including executing your code (simulation) and launching third party tools (simulator, synthesizers).
2. The navigation bar helps you navigate through your project and open files for editing. You can move back and forth between file without having to reopen and reschroll between files. You also have access to shorcuts to run a synthesis or a simulation.
3. The editor window is where you create and modify code. Depending on the current file type, the editor can change. For example, when editing a Cx file, the editor displays the Cx editor. When editing a Verilog file, the editor display a textual editor.
4. The network view displays how tasks are connected together and forms a network. The network view is activated only when you edit Cx netowrk files.
5. The project window gives you access to all your projects and files. You can expand them, collapse them, or close then when necassary.
6. The FSM view displays the [Finite State Machine](https://en.wikipedia.org/wiki/Finite-state_machine) created from your Cx code. The FSM view is activated only when you edit a Cx file that generates a FSM.
7. The Git window helps you work in team. You can commit, push, pull your code on a shared or private repository using this version control system (VCS).
8. The Status bar displays the status of your project and the IDE itself, as well as any warnings or messages.
9. The tool window bar is where you can see various information such as warnings, errors, messages, but also logs from third party tools during compilation and synthesis.

You can organize the main window to give yourself more screen space by hiding or moving toolbars and tool windows. You can also use keyboard shortcuts to access most IDE features.

### Tool Windows

For maximum user-friendliness, the Synflow IDE offers a preset perspectives that you can modify. By default, the most commonly used tool windows are availables at the edges of the application window.

- To Maximize or Minimize a tool window, directly click the icon in the window bar. You can also drag, pin, unpin, attach, and detach tool windows.
- To return to the current default tool window layout, click **Window > Perspective > Reset Perspective ** or save your own layout by clicking  **Window > Perspective > save perspective**.
- To focus on a specific window double click on the window bar (double click again to go back to your perspective).
- To add a new tool window, select **Window > show view** and add another tool window.

You can use *Quick Access* to search and filter within most tool windows in Synflow IDE. Quick Access also helps you coding. For more tips, see [Keyboard Shortcuts](https://developer.android.com/studio/intro/keyboard-shortcuts.html).

### Code Completion

Synflow IDE offers the code completion feature, which you can access using keyboard shortcut CTRL+Space. This feature displays relevant options based on the context for variables, types, methods, expressions, and so on. It can also completes the current statement for you, adding missing parentheses, brackets, braces, formatting, etc.

You can also perform quick fixes and show intention actions by pressing CTR+1.

For more information about code completion and quick fix, see the Eclipse website on [Code Completion](http://www.eclipse.org/recommenders/getting-started/) and [Quick Fix](http://help.eclipse.org/neon/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2Fconcepts%2Fconcept-quickfix-assist.htm).

### Navigation

Here are some tips to help you move around Synflow IDE.

- Switch between your recently accessed files using the *back, forward, Last Edit arrows*. Press CTRL+Q to bring up the last file you edited, ALT+Left to bring up the last file you viewed, and ALT+Right to go back to where you were. You can also click on the arrows.
- View the structure of the current file using the *down arrow* on the left of your file. The File Structure shows you a summary of the insight of your current file and you can quickly navigate to any part of your code.
- Search for and navigate to a specific task or netowrk in your project using the *Quick Access* action. Bring up the action by pressing CTRL+3. Navigate to anything, including tasks, expressions, statements, etc.
- Clean a project (when you want to regenerate your files) using the *clean* action. Bring up the clean action by pressing ALT+P+N and select the project(s) you want to clean.
- Run an application (simulation or synthesis) using the *Run* action. Bring up the Run action by pressing CTRL+F11 and select an application (if you alreay ran an application before, it will run it again) or right click on your network and select Run.
- Configure the configuration of a Run action using the *Run as* action. Bring up the Run as action by pressing ALT+R,N and configure the execution of a Run action.

### Style and Formatting

As you edit, the Synflow IDE automatically applies formatting and styles as specified in the Default Format Settings. You can also explicitly call the *Reformat Code* action by pressing CTRL+SHIFT+F.

![Code before formatting.](/images/synflow/codeUnformat.png "Code before formatting.")

![Code after formatting.](/images/synflow/codeFormat.png "Code after formatting.")

### Version Control Basics

Synflow IDE supports a variety of version control systems, including Git, GitHub, CVS, and Subversion.

Git is widely used by developers so it is natively integrated in the IDE. Other version control systems can be installed as an alternative to the Git versionning system.

After cloning your application or project into the Synflow IDE, use the Git window and Git options to share your code and work in team. You can create a repository, import the new files into version control, and perform other version control operations.

![The Git window and menu.](/images/synflow/gitWindow.png "The Git window and menu.")

## Build System

The Synflow IDE uses a leading-edge compiler as the foundation of the build system. This compiler compiles the Cx code into **synthesisers friendly, readable, Verilog or VHDL code**. This is vitally important to get the best Quality of Results when synthesising your application for a specific FPGA (or for an ASIC). Indeed, all synthesizers need the code to follow specific coding rules to be able to optimize the performance of an application. If the coding rules are violated, it has a very negative impact on the performance.

This is a new, pragmatic, and concrete approach to generating hardware code from higher level of abstraction languages. Competitive products focus on generating RTL code from pseudo SystemC or C code while neglecting the readibility of the compiled code and the Quality of Result. We are doing the opposite because we beleive that performance matters and that engineers are capable of learning new languages in no time provided that they are simple and modern.

The build system runs as an integrated tool from the Synflow IDE menu. You can use the features of the build system to do the following:

- Generate VHDL or Verilog code.
- Generate simulation files.
- Generate preconfigured files.
- Call third party tools and run a simulation or a synthesis.
- Create multiple configuration for your project, with different targets using the same project and modules.
- Reuse code and resources across various projects.

By employing the flexibility of the compiler, you can achieve all of this without modifying your project core source files. Synflow IDE compiled files has the same name as the original file. They are plain text Verilog or VHDL files. Each project also has one build file that describe the compilation options. When you execute an application, the Synflow IDE automatically generates the necessary build file.

To learn more about the build system and how to configure, see [Configure Your Build](https://developer.android.com/tools/building/plugin-for-gradle.html#build-config).

### Build Variants

The build system can help you create different configuration of Run action for the same application from a single project. This is useful when you have multiple targets (FPGAs) or if you want to specify different compilation options for different device configurations.

![A configuration for one of the projects.](/images/synflow/runConfig.png "A configuration for one of the projects.")

For more information about configuring build variants, see [Configuring Gradle Builds](https://developer.android.com/tools/building/configuring-gradle.html#workBuildVariants).

### Resource Shrinking

The Synflow IDE automatically skip unused resources from your project. For example, you can add Cx files, Verilog files, or any other files for references on your project, and the compiler will automatically ignore them when he build your application.

### Managing Dependencies

Dependencies for your project are specified by name in the `Project Reference` window. The compiler takes care of finding your dependencies and making them available in your build. To add a depency to your project, selecte your projet and press ALT+Enter or right click on your project and select Properties. In project properties, select your dependency(ies).
  
![Selecting properties for a project.](/images/synflow/selectProperties.png "Selecting properties for a project.")

![Adding the dependency(ies).](/images/synflow/projectDependencies.png "Adding the dependency(ies).")

For more information about configuring dependencies, read [Configure Build Variants](https://developer.android.com/tools/building/configuring-gradle.html#dependencies).

## Debug Tool

The Synflow IDE assists you in debugging and improving the performance of your code by offering a **fast and efficient simulator and a code checker**. The Synflow simulator can execute your code up to 20 times faster than traditional RTL simulators while offering the same accuaracy (it is cycle accurate and bit accurate). The code checker controls your code as you write to detect errors. You can simulate a network or a task depending on your needs.

### Synflow Simulator

To Run a simulation, right click on a network and select **Run as > Simulation**. The code will automatically be compiled and executed.

![Running a simulation.](/images/synflow/simulation.png "Running a simulation.")

To stop a simulation, in the **tool window** bar, click the square red button. 

For more information about the Synflow simulator, see [Android Monitor](https://developer.android.com/tools/help/android-monitor.html).

### Code checker

The code checker is always active. When you save your file after editing the code, it will control your code. If an error is detected, you can modify your code yourself or use **Quick Fix**.

![An error detected on the code.](/images/synflow/errorCatcher.png "An error detected on the code.")
  
For more informatin about the code checker, see [Dumping and Analyzing the Java Heap](https://developer.android.com/tools/help/am-memory.html#dumping).

### Third party Simulator

You can execture a thrid party simulator from Synflow to run an RTL simulation of the generated code. For that purpose, you need [Mentor Graphics Modelsim](https://developer.android.com/tools/help/android-monitor.html) installed on your computer and the Synflow IDE correctly configured. To run an RTL simulation from the Synflow IDE, select **Run Configuration** or type ALT+R,N. Check **use Modelsim** in the simulation option, and hit **Run**.

![Running a simulation with Modelsim.](/images/synflow/modelsimSimulation.png "Running a simulation with Modelsim.")

You can see the result on Modelsim or in the **tool window** bar (**Console View**).

For more information about using a third party simulator, see [Android Monitor](https://developer.android.com/tools/help/android-monitor.html).

### Annotations in Synflow IDE

Synflow IDE supports annotations for Finite State Machine to help you understand the cycle accurate behavior of your code and catch bugs, such as unsyncrhonized data between tasks. You can annote all states of an FSM or just a few.

    rxTask = new task {
      out bool start;
    
      u8 data = 0;
      u4 i; 
      u6 j;
    
      void loop() {
        detectStart: rxFilter.startRx.read; // Detect the start bit 
        
        waitData: for (j = 0; j != OVERSAMPLING + OVERSAMPLING / 2; j++) { // Wait for the data 
          prescaler.tick.read;
        }
    
        saveData: for (i = 0; i != 8; i++) { // read data (LSB)
          data >>= 1;
          data[7] = rxFilter.filteredRx.read();
          waitOverSamp: for (j = 0; j != OVERSAMPLING; j++) { // Wait for the next bit
            prescaler.tick.read;
          }
        }
    
        if (rxFilter.filteredRx.read()) {
          dout.write(data);
          print("UART Rx data: ", data);
        }
      }
    };

You will see the name of your sates in the FSM view and while running a simulation.

![FSM view with annotations.](/images/synflow/fsmView.png "FSM view with annotations.")

### Log messages

When you build and run your project with the Synflow IDE, you can view the output and log messages by clicking **Console** in the **tool window** bar. You can also view errors and warning on all your opened projects.
  
![Errors and Warnings.](/images/synflow/errrorsWarnings.png "Errors and Warnings.")
