# Intro to Edge AI: Embedded Linux Development and Traditional Computer Vision Techniques
A workshop to teach beginner embedded developers how to set up a SBC with linux, the basics of the Command Line, Python enviroments, OpenCV and Deep Learning Detection Models

      ____     .--.               _____ 
     | !? |   |o_o |             < OOM >
      `--'    |:_/ |              ----- 
        \    //   \ \                 \   ^__^
            (|     | )                 \  (oo)\_______
           /'\_   _/` \                   (__)\       )\/\
           \___)=(___/                        ||----w |
                                              ||     ||

Welcome to the foundational workshop! Today, we are going to turn a single-board computer into an Edge AI device capable of seeing the world using traditional computer vision algorithms.

**Hardware Requirement:** A Raspberry Pi 4 (or newer) and a standard USB webcam.

## Part 1: Booting Up the Edge

### What is a Raspberry Pi and Edge AI?
A Raspberry Pi is a Single Board Computer (SBC)—an entire, fully functioning computer packed onto a board the size of a credit card. It runs an embedded Linux operating system. Embedded Linux is the hidden engine of the modern world, powering everything from smart thermostats to the entertainment screens on airplanes.

Traditionally, AI requires massive, centralized servers in giant data centers. But what if a drone needs to navigate a forest without internet access? That's where **Edge AI** comes in. By running AI models directly on the "edge" of the network (on the device itself), we reduce latency, increase privacy, and save bandwidth. 

**Learning Objectives:**
1. Provision and remotely access a headless embedded Linux system.
2. Master fundamental command-line interface (CLI) navigation and scripting.
3. Establish a modern Python virtual environment to run an OpenCV computer vision pipeline.

### Step 1: Create a Raspberry Pi Connect Account
Later on, we will need to see the graphical output of our computer vision scripts. Go to [connect.raspberrypi.com](https://connect.raspberrypi.com/) and create a free account. Keep your login details handy.

### Step 2: Flashing the OS
1. Insert your micro SD card into your computer.
2. Open the **Raspberry Pi Imager**.
3. Select the recommended **Raspberry Pi OS (64-bit)** (Full desktop version).
4. Select your SD card as the storage.
5. **CRUCIAL STEP:** Before clicking write, edit the OS Customization settings!
   
   * Set a hostname.
   * Set a username and password (write these down!).
   * Configure your Wi-Fi (if not using ethernet).
   * Go to the Services tab and **Enable SSH** (use password authentication).
   * Go to the Options tab and enable Raspberry Pi Connect.
6. Write the image to the SD card.

### Step 3: Finding Your Pi on the Network
Before GUIs (Graphical User Interfaces) with mice and windows existed, humans interacted with computers using a **terminal**—a text-based interface where you type commands directly to the operating system. We are going to use a terminal to find our Pi.

1. Open **Command Prompt** (Windows) or **Terminal** (Mac).
2. Run this command to see a list of all IP addresses currently on your local network:
   `arp -a`
3. Physically put the micro SD card into the Raspberry Pi. Plug in the Ethernet cable, and finally, plug in the power. 
4. Wait about 2-3 minutes for the Pi to boot up.
5. Run `arp -a` again. Compare the new list to the old list. The new IP address that appeared is your Raspberry Pi!
   * *Fallback:* If this doesn't work, log into Raspberry Pi Connect in your browser, open the remote terminal, and type `ifconfig` to find the `inet` IP address under your network adapter.

### Step 4: Connecting via SSH
We need that IP address to **SSH** into the Pi. Secure Shell (SSH) is a network protocol that gives you a secure, encrypted way to access a computer over an unsecured network. It drops you into a **shell**—a program that takes your keyboard commands and passes them to the operating system to execute.

Run this command, replacing `username` with your chosen username and `ip_address` with the IP you found:
`ssh username@ip_address`

Type `yes` to accept the fingerprint, and enter your password.

### Step 5: The First Command
You are now inside your Raspberry Pi! You are currently sitting in your **home directory** (`~`), your personal workspace. Let's update the system. Run:
`sudo apt-get update && sudo apt-get upgrade -y`

* **`apt-get`**: This is the Advanced Package Tool, Linux's built-in app store/package manager.
* **`sudo`**: "Superuser Do". It temporarily gives you administrative (root) privileges to install things safely.

---

## Part 2: Taming the Command Line

Let's learn how to navigate. 
* **`mkdir`** (Make Directory): Creates a new folder.
* **`cd`** (Change Directory): Moves you into a folder.
* **`ls`** (List): Shows you what is inside your current folder.

### Step 1: Installing a Superpower (Vim)
Before we start making files, let's install a better text editor. We will use `apt-get` to install programs in Debian-based Linux operating systems like Raspberry Pi OS. 

Run this command:
`sudo apt-get install vim -y`

*What is Vim?* It is an upgraded version of `vi`, one of the oldest text editors which, along with `nano`, comes preinstalled on almost all Linux distributions. Vim is notoriously a bit esoteric and relies heavily on keyboard commands instead of a mouse. It is a long-running joke in the software world that the hardest part of programming is simply figuring out how to exit Vim! 

To save you from that trap, you need to understand Vim's **two main modes**:

* **Normal Mode:** You start here. You cannot type standard text. Instead, your keyboard acts as a command board. Use your arrow keys to move your cursor around the file. 
* **Insert Mode:** Press the `i` key to enter this mode. Now you can type text normally just like in Notepad or Word.
* **The Escape Hatch:** To stop typing and go back to Normal Mode, press the `Esc` key.
* **How to Exit:** While in Normal Mode, type `:wq` and hit `Enter` to **w**rite (save) and **q**uit. If you made a mistake and want to panic-exit without saving, type `:q!` and hit `Enter`.

### Step 2: Setting up the Workspace
Create a development folder and clone our workshop repository:
`mkdir dev`
`cd dev`
`git clone https://github.com/bevis-hp/Intro-to-Embedded-Linux-Development-Computer-Vision-and-Edge-AI.git`

Go back to your home folder (`cd ~`) and create a folder for binary executables:
`cd ~`
`mkdir bin`

### Step 3: The Mystery Script
Move the bash script from the workshop folder into your new `bin` folder (`cp` copies files):
`cp ~/dev/Intro-to-Embedded-Linux-Development-Computer-Vision-and-Edge-AI/mystery_script.sh ~/bin/`

Right now, it's just a text file. We need to make it an executable program using **`chmod`** (Change Mode):
`cd ~/bin`
`chmod +x mystery_script.sh`

Run it! Notice we use `./` to tell the computer "run the file in *this exact folder*":
`./mystery_script.sh`

It prints an ASCII cat and asks for a text file. Let's rename the script using **`mv`** (Move/Rename):
`mv mystery_script.sh devcat`

### Step 4: Modifying the PATH
We want `devcat` to run from anywhere, without needing the `./`. We do this by adding our `bin` folder to the system's `$PATH`.
1. Open your bash configuration file with the **`vim`** text editor:
   `vim ~/.bashrc`
2. You are in Normal Mode. Press `G` (capital G) to jump to the bottom of the file.
3. Press `o` (lowercase o) to open a new line below your cursor and automatically enter Insert Mode.
4. Type exactly: `export PATH="$HOME/bin:$PATH"`
5. Press `Esc` to exit Insert Mode and return to Normal Mode.
6. Type `:wq` and hit `Enter` to write the file and quit.

Reload the configuration using **`source`** and verify it with **`echo`**:
`source ~/.bashrc`
`echo $PATH`

### Step 5: Testing DevCat
Go to your home directory (`cd ~`). Let's create an empty text file using **`touch`**:
`touch test.txt`

Open it with `vim test.txt`. Press `i` to enter Insert Mode, write a sentence, press `Esc` to return to Normal Mode, and type `:wq` to save and quit. 

Now, run your command from anywhere!
`devcat test.txt`

Check the file again with `cat test.txt`. The original text is gone, replaced entirely by cat language! *(The **`cat`** command, short for "concatenate," is most commonly used to quickly read a file and print its contents directly to your terminal screen).*

### Step 6: The Ultimate Cheat Sheet (The `man` Command)
Before you rely on Google for command line help, you should know that Linux comes with a built-in instruction manual for almost everything. It is accessed using the **`man`** (manual) command.

Try typing this:
`man ls`



This opens the official documentation for the `ls` command. You navigate this exactly like Vim: use your arrow keys to scroll up and down to read about all the different flags. When you are done reading, press **`q`** to quit.

### Further Practice: Honing Your Developer Skills
If you finish the workshop early, check out these excellent interactive resources:

**Command Line & Text Editing Games**
* **Vim Adventures** ([vim-adventures.org](https://vim-adventures.org/)): An online Zelda-style puzzle game to learn real Vim keyboard commands.
* **The Command Line Murders** ([github.com/veltman/clmystery](https://github.com/veltman/clmystery)): An interactive mystery where you play a detective trying to solve a crime using terminal commands (`grep`, `cat`, `ls`, etc.).
* **OverTheWire: Bandit** ([overthewire.org/wargames/bandit](https://overthewire.org/wargames/bandit/)): A legendary "wargame" for beginners. Use SSH to log into a remote server and locate hidden passwords using Linux commands.

**Essential Developer Resources**
* **ExplainShell** ([explainshell.com](https://explainshell.com/)): Paste any confusing Linux command into the search bar to visually break down what every flag and argument does.
* **tldr pages** ([tldr.sh](https://tldr.sh/)): Community-driven simplified examples of how to use commands (`sudo apt-get install tldr`).
* **Linux Journey** ([linuxjourney.com](https://linuxjourney.com/)): A beautifully designed, module-by-module guide to mastering Linux.
* **Learn Git Branching** ([learngitbranching.js.org](https://learngitbranching.js.org/)): The most visual and interactive way to learn Git version control on the web.

---

## Part 3: Computer Vision Fundamentals (Sobel Filter)

**Clarification Time: Computer Vision vs. Edge AI**
Before we get to Artificial Intelligence, we need to understand standard Computer Vision (CV). Computer Vision uses mathematical algorithms and matrix calculations to process images. In this section, we will do *image edge detection*—which means mathematically finding the physical outlines of objects in a picture. 

*Do not confuse this with Edge AI!* "Edge AI" refers to where the computing happens (on a local *edge device* like our Pi), while "image edge detection" refers to finding the borders of objects in a photo. 

Because we are working with visual media, we need a desktop environment. 

1. Go back to [connect.raspberrypi.com](https://connect.raspberrypi.com/).
2. Connect to your Raspberry Pi using the **Screen Sharing** (Desktop) option.
3. Open the Terminal app inside the desktop GUI.

### Step 1: Python and Virtual Environments
When developing in Python, it's best practice to use **Virtual Environments**. This creates an isolated bubble for your project, preventing different Python projects from messing up each other's libraries. We will use a lightning-fast modern tool called `uv`.

Install `uv` using **`curl`** (a command that downloads data from a server and pipes it to an execution shell):
`curl -LsSf https://astral.sh/uv/install.sh | sh`
`source $HOME/.local/bin/env`

### Step 2: Setting up OpenCV Workspace
Instead of making a new folder, we will set up our environment directly inside our cloned repository. 
`cd ~/dev/Intro-to-Embedded-Linux-Development-Computer-Vision-and-Edge-AI`
`uv venv`

Activate the environment and install OpenCV:
`source .venv/bin/activate`
`uv pip install opencv-python numpy`

### Step 3: Preparing the Assets
Let's download an image from the internet using `curl` and output (`-o`) it as `test_image.jpg`:
`curl -o test_image.jpg https://raw.githubusercontent.com/opencv/opencv/master/samples/data/ml.png`

### Step 4: Inspecting the Code
Open the Python script with `vim sobel_edge.py`. Ensure the `target_image` variable matches the file you just downloaded (`test_image.jpg`). 

Before we run it, let's look at how the OpenCV library (`cv2`) processes the image mathematically:
* **`cv2.cvtColor(source, cv2.COLOR_BGR2GRAY)`:** This strips the color data. Finding physical image edges is about contrasting light and dark pixels; color just wastes memory.
* **`cv2.Sobel(...)`:** This is the core math. It calculates the *derivative* (the rate of change) of pixel intensity. `sobel_x` looks for vertical lines (rapid changes left-to-right), and `sobel_y` looks for horizontal lines (rapid changes top-to-bottom).
* **`cv2.addWeighted(...)`:** This function takes our isolated horizontal and vertical lines and blends them back together into a single, cohesive image.



Type `:wq` to save and quit.

### Step 5: Run the Filter
Run your python script:
`python sobel_edge.py`

A window should pop up showing the original image alongside a mathematically transformed image highlighting all the object outlines! 

---

## Part 4: Computer Vision with Live Video

Now that we can process static images, let's process real-time video using a standard USB webcam. 

### Step 1: Connect and Identify the Camera
Plug your USB web camera into one of the Raspberry Pi's USB ports. 

To use our camera, we need to find its hardware device ID. We will use `apt-get` to install a small utility package that helps list connected video devices. Run:
`sudo apt-get install v4l-utils -y`

Now, list all connected video devices:
`v4l2-ctl --list-devices`

You will see your camera's name listed along with several `/dev/video` endpoints beneath it. Look for the very first `/dev/video` number listed under your camera's name (usually `/dev/video0`). That number (`0`) is your camera ID.

### Step 2: Inspecting the Code
Ensure you are still in your repository directory and your virtual environment is active. Let's look inside the script using our text editor:
`vim usb_camera_test.py`

Check the `camera_id` variable near the top of the file. If your camera ID from Step 1 was anything other than `0`, press `i` to enter Insert Mode, change the number, and press `Esc`.

Let's look at how we capture live video:
* **`cv2.VideoCapture(camera_id)`:** This tells OpenCV to take control of the physical USB hardware.
* **`while True:`:** Video is just a rapid series of static images. This creates an infinite loop that runs as fast as the Pi's processor allows.
* **`video_capture.read()`:** Inside the loop, this grabs the absolute latest frame from the camera sensor. 
* **`cv2.waitKey(1)`:** This tells the GUI window to pause for 1 millisecond to refresh the display, and simultaneously listens to see if you pressed the 'q' key to break the loop.

Type `:wq` to save and quit (or `:q` if you made no changes).

### Step 3: Run the Live Feed
Execute your script:
`python usb_camera_test.py`

A new window should open on your Raspberry Pi desktop showing a live video feed from your webcam. Make sure the video window is actively selected and press the `q` key on your keyboard to close it.

---

## Part 5: Real-Time Sobel Filters

We have successfully processed a static image, and we have successfully streamed live video. Now, let's combine them. We are going to apply the algorithmic Sobel filter to every single frame of our live video feed. 

### Step 1: Inspecting the Code
Let's open it up in Vim:
`vim live_sobel.py`

Look closely at the `while True:` loop. The architecture of a real-time vision pipeline is simply **Read -> Process -> Display**.
1.  **Read:** We grab `video_frame` using `video_capture.read()`.
2.  **Process:** Instead of immediately showing it, we pass that frame through the exact same `cvtColor`, `Sobel`, and `addWeighted` functions we used in Part 3.
3.  **Display:** We show the newly generated `combined_edges` frame via `cv2.imshow()`.

*(Note: Change the `camera_id` variable if needed, then type `:wq` or `:q` to exit).*

### Step 2: Run the Edge Stream
Execute your script:
`python live_sobel.py`

You should now see two windows: your standard live feed, and a real-time feed consisting entirely of calculated lines! Press `q` while selecting one of the video windows to close the program.

### What's Next?
Applying math to every single pixel of a 30fps video is quite computationally intensive. Now that we have a live video feed established, we can use simpler, more efficient Computer Vision techniques to compare frames against each other over time. This allows us to do **motion detection**, which leads us directly into Part 6...

---

## Part 6: Computer Vision: Motion Detection

Instead of analyzing every frame from scratch, we can take a "baseline" photo of the static background. Then, for every new video frame, we mathematically subtract the new frame from the baseline. If the result is greater than zero, those specific pixels must represent movement! 

### Step 1: Inspecting the Code
`vim motion_detect.py`

This script introduces some vital new concepts:
* **`time.sleep(2)`:** When a webcam turns on, its auto-exposure and white-balance need a moment to settle. If we grab our baseline frame too early, a sudden lighting adjustment will make the script think the entire room is moving. 
* **`cv2.GaussianBlur(...)`:** Cameras produce "noise" (tiny static artifacts). Blurring the image smooths out this noise so it isn't falsely flagged as motion.
* **`cv2.absdiff(baseline, current_frame)`:** This function finds the absolute difference between the empty room and the current room. 
* **`cv2.threshold(...)`:** This takes any pixel that changed and forces it to become stark white (`255`), turning everything else pure black (`0`). This creates a "binary mask".
* **`cv2.findContours(...)`:** This function scans our binary mask and draws an invisible outline around any blob of white pixels, allowing us to generate our green bounding boxes!



*(Note: Remember to change the `camera_id` variable if needed, then type `:wq` to save and quit).*

### Step 2: Run the Motion Tracker
**Important:** Before you run this command, point the camera at a static background and *step out of the frame*. 

Execute your script:
`python motion_detect.py`

Two windows will pop up: The live color feed with a tracking box, and a black-and-white window showing the threshold math in action. Press `q` to safely exit the program.

---

## Next Steps: Choose Your Path!

Congratulations on completing the foundations of Embedded Linux and Computer Vision! You now have a solid understanding of how to process pixels mathematically. 

However, motion detection only tells us *where* something is, but it doesn't tell us *what* it is. A waving tree branch triggers our last script just as easily as a person. To solve this, we are finally ready to step out of traditional algorithms and introduce true Edge AI.

Depending on your interests, you can now move on to any of the following advanced modules:
* **Intro to Edge AI: Object Detection and Recognition** *(Requires Raspberry Pi 5)* - Classify real-world objects using deep learning neural networks.
* **Intro to Edge AI: Hand Tracking and Physical Computing Applications** - Map human gestures to physical hardware actions.
* **Intro to Edge AI: LLMs and Voice Recognition** - Build conversational interfaces that run entirely offline.
