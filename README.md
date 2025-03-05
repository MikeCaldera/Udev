# Udev
Udev in Proxmox Debian

Here is a help file written in a clear, concise, and user-friendly manner to explain the concept of ACTION=="add|change" in udev rules. It avoids overly technical jargon where possible and uses relatable examples to make the reasoning accessible.

# Understanding ACTION=="add|change" in udev Rules
This help file explains why we use ACTION=="add|change" instead of just ACTION=="add" in udev rules for managing disk symlinks (e.g., linking a disk’s serial number to a name like disk00). It’s especially useful in setups with lots of disks, like servers or storage arrays.

# What’s a udev Rule?
A udev rule is like a set of instructions for your computer’s operating system (Linux). It tells the system what to do when it detects hardware—like a hard drive or SSD—such as giving it a consistent name (a symlink) based on its serial number.
For example:

ACTION=="add|change", SUBSYSTEM=="block", ENV{DEVTYPE}=="disk", ENV{ID_SCSI_SERIAL}=="xxx", SYMLINK+="disk00"

This rule says: "When a disk with serial number xxx is detected or updated, call it disk00."
What Does ACTION Mean?
The ACTION part tells udev when to apply the rule, based on what’s happening with the device:

    add: A new device is plugged in or detected (e.g., you connect a USB drive or reboot the system).
    change: An existing device’s details—like its name or status—are updated without unplugging it.

Using ACTION=="add|change" means the rule runs for both situations.
# Why Not Just Use add?
Using only ACTION=="add" works fine if your disks never change after they’re plugged in. But in real-world setups—like a server with dozens of disks—things aren’t always that simple. Here’s why add alone can miss the mark:
1. Disks Can Shift After Boot

    During a reboot, the system might name a disk sdi one time and sdaq the next. If the rule only runs on add (the first detection), it might not catch this shift, leaving your symlink (disk00) pointing to the wrong disk—or nowhere at all.

2. Rescanning Doesn’t Always “Add”

    Sometimes you tell the system to recheck its disks (e.g., after adding a new one to a storage array). This is called a “rescan.” The system updates the disk’s info but doesn’t treat it as a new device—so no add event happens. Without change, the rule won’t update the symlink.

3. Connection Hiccups

    If a disk briefly disconnects and reconnects (e.g., a loose cable or a controller glitch), the system might not see it as a fresh add. Instead, it sends a change event to update the disk’s status. With only add, the symlink could get lost.

# Why add|change Is Better
Using ACTION=="add|change" covers all the bases:

    On add: The rule runs when a disk is first detected—like plugging in a drive or booting up.
    On change: The rule runs again if the disk’s details update—like a rescan or name change.

This ensures your symlink (e.g., disk00) always points to the right disk, no matter what happens.
Real-Life Example
Imagine you have a disk with serial number 96K0A154FT3C1638 that’s supposed to be called disk00:

    Before reboot: It’s named sdi, and disk00 links to sdi.
    After reboot: The system calls it sdaq instead.
    With add only: The rule might not fire (since the disk isn’t “new”), and disk00 could break.
    With add|change: The rule catches the update and fixes disk00 to point to sdaq.

Later, if you rescan the disk array (e.g., to find a new drive), change ensures disk00 stays correct even if the disk’s name shifts again.
When Does This Matter Most?
This combo is a lifesaver in setups with:

    Lots of disks: Like a server with 92 drives, where names like sda, sdb, etc., can shuffle around.
    Hotplugging: Disks added or removed without rebooting.
    Rescans: When you manually refresh the disk list.

For a simple home PC with one drive, add might be enough. But for bigger, dynamic systems, add|change keeps things reliable.
How We Use It
In our setup:

    90-SSD.rules: Links SSDs to names like OSDisk01 or SSDPoolDiskXX based on their serials.
    91-DAS.rules: Links storage array disks to disk00 through disk83, tied to their slot numbers.

# Using add|change ensures these names stick, even if the system renames disks or updates their info.
Key Takeaway

    add alone: Works for the first detection but misses updates.
    add|change: Covers both new disks and changes, keeping symlinks rock-solid.

Think of it like labeling a seat at a table: add assigns the label when someone sits down, but change fixes it if they switch chairs later. With add|change, your labels always match the right person—er, disk!
This help file should be easy to follow for someone new to udev rules while still providing enough detail for practical understanding. Let me know if you'd like adjustments!
