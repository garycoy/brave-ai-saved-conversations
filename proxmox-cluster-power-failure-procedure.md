I have a 3 node proxmox cluster that can detect when it loses power. Each node on the cluster has a VM that I run docker containers from. (eg proxmox1,proxmox2,proxmox3 running docker1,docker2,docker3). I have shell script in each physical node running and checking every 30 seconds to see if it is on power or battery. When battery reaches 50%, I want to initiate a clean shutdown of everything. I have several docker containers in the docker vms. I have a postgresql database in one container on proxmox3. Several apps/containers run against the postgresql db. How to best shutdown this configuration/system?
To safely shut down your Proxmox cluster with a PostgreSQL database during a power failure, you need a coordinated shutdown sequence that prioritizes data integrity. Since your database is on proxmox3, the order of operations is critical to prevent corruption.

Here is the recommended strategy:

Stop Application Containers First: On proxmox1 and proxmox2, stop all Docker containers that connect to the PostgreSQL database. This ensures no new writes are attempted while the shutdown begins.
Stop Database Container: On proxmox3, stop the PostgreSQL Docker container. This allows the database to flush buffers and close connections cleanly.
Shutdown Docker VMs: Shut down the VMs running the Docker instances (docker1, docker2, docker3) in a graceful manner (e.g., qm shutdown or virsh shutdown), allowing the guest OS to stop remaining services.
Shutdown Proxmox Nodes: Finally, shut down the physical Proxmox nodes themselves.
Recommended Script Logic
You should modify your existing shell scripts to handle this sequence. Since the nodes are independent, you might want a "master" logic or a shared script that runs on all nodes but behaves differently based on the node name.

Key considerations for your script:

Communication: If proxmox1 and proxmox2 need to know about the state of proxmox3, you may need a shared file on a network mount (like NFS) or a simple HTTP check. However, a simpler approach is to have the script on each node check its own battery and only act on local resources, except for the node holding the database.
The Database Node Priority: The script on proxmox3 must run the database shutdown step before shutting down the VMs.
Example Logic for proxmox3 (The DB Node):

bash

Copy
#!/bin/bash

# Check battery (simplified logic)
BATTERY=$(upower -i /org/freedesktop/UPower/devices/battery_BAT0 | grep percentage | awk '{print $2}' | tr -d '%')

if [ "$BATTERY" -le 50 ]; then
    echo "Low battery ($BATTERY%). Initiating clean shutdown sequence on proxmox3..."

    # 1. Stop the PostgreSQL container
    docker stop postgres_container_name

    # 2. Stop other local containers if any
    docker stop other_local_containers

    # 3. Shutdown the Docker VM (replace 'docker3' with actual VM ID or name)
    # Using qm shutdown for graceful guest OS shutdown
    qm shutdown 103 --timeout 300 

    # 4. Wait for VM to stop (optional, but safer)
    sleep 60

    # 5. Shutdown the Proxmox node itself
    shutdown -h now
fi
Example Logic for proxmox1 and proxmox2 (App Nodes):

bash

Copy
#!/bin/bash

BATTERY=$(upower -i /org/freedesktop/UPower/devices/battery_BAT0 | grep percentage | awk '{print $2}' | tr -d '%')

if [ "$BATTERY" -le 50 ]; then
    echo "Low battery ($BATTERY%). Stopping application containers on proxmox1/2..."

    # 1. Stop all containers that connect to the DB
    # You might tag these containers with a specific label or name pattern
    docker stop $(docker ps -q --filter "name=app_container")

    # 2. Shutdown the Docker VM
    qm shutdown 101 --timeout 300 # Adjust ID for proxmox1

    # 3. Shutdown the Proxmox node
    shutdown -h now
fi
Critical Recommendations
Timeouts: Ensure the qm shutdown command has a sufficient timeout (e.g., 300 seconds) to allow the guest OS to stop Docker and flush PostgreSQL logs.
Network Dependency: If your script relies on checking the status of other nodes (e.g., "wait for proxmox3 to finish"), ensure the network remains up long enough. If the UPS cuts power to the network switch immediately, the nodes might not be able to talk to each other. Independent shutdown logic (as shown above) is usually safer in this scenario.
PostgreSQL pg_ctl: If you have direct access to the DB inside the container, ensure the container is configured to handle the SIGTERM signal gracefully to trigger a checkpoint before exiting.
Testing: Simulate a power loss (unplug the UPS) in a test environment to verify the order of operations and ensure the database doesn't crash.
Would you like a more detailed script that includes error handling or checks for specific container names?






Qwen VL 235B
I currently have the 'power checking/ups checking' script in the proxmox physical host. How to communicate to the docker VMs that it's time to cleanly shutdown?
Since your power-checking script runs on the Proxmox host (the physical node) and the Docker containers run inside VMs, the host needs to send a signal to the guest OS to trigger the shutdown sequence.

The most reliable method in a Proxmox environment is to use the qm command to send a graceful shutdown signal to the VMs. This relies on QEMU Guest Agent being installed and running inside the VMs.

Here is the best approach to communicate the shutdown:

1. Prerequisite: Enable QEMU Guest Agent
For qm shutdown to work gracefully (sending an ACPI signal to the OS to stop services properly) rather than forcing a hard power-off, you must ensure the QEMU Guest Agent is installed and enabled in each Docker VM.

Inside the VM (Ubuntu/Debian example):
bash

Copy
sudo apt-get install qemu-guest-agent
sudo systemctl enable qemu-guest-agent
sudo systemctl start qemu-guest-agent
In Proxmox Web UI: Go to the VM -> Options -> Ensure QEMU Guest Agent is set to Enabled.
2. The Host-Side Script Logic
Your script on the Proxmox host should follow this specific order to ensure data integrity:

Stop App VMs First: Send shutdown signals to docker1 and docker2 (the app nodes).
Wait: Give them time to stop their containers and close DB connections.
Stop DB VM: Send shutdown signal to docker3 (the Postgres node).
Shutdown Host: Finally, shut down the physical Proxmox node.
3. Example Script for Proxmox Host
Save this script on your Proxmox host (e.g., /usr/local/bin/ups-shutdown.sh) and make it executable (chmod +x).

bash

Copy
#!/bin/bash

# Configuration
UPS_BATTERY_THRESHOLD=50
VM_APP_1="101"   # Replace with actual VM ID for docker1
VM_APP_2="102"   # Replace with actual VM ID for docker2
VM_DB="103"      # Replace with actual VM ID for docker3 (Postgres)
TIMEOUT_APP=300  # Seconds to wait for app VMs to stop
TIMEOUT_DB=300   # Seconds to wait for DB VM to stop

# Check Battery (Adjust path based on your hardware, e.g., /sys/class/power_supply/BAT0)
# Example for generic laptop/UPS detection:
BATTERY_PERCENT=$(upower -i /org/freedesktop/UPower/devices/battery_BAT0 2>/dev/null | grep percentage | awk '{print $2}' | tr -d '%')

# Fallback if upower fails (check /sys/class/power_supply directly)
if [ -z "$BATTERY_PERCENT" ]; then
    if [ -f /sys/class/power_supply/BAT0/capacity ]; then
        BATTERY_PERCENT=$(cat /sys/class/power_supply/BAT0/capacity)
    fi
fi

echo "Current Battery: ${BATTERY_PERCENT}%"

if [ -n "$BATTERY_PERCENT" ] && [ "$BATTERY_PERCENT" -le "$UPS_BATTERY_THRESHOLD" ]; then
    echo "CRITICAL: Battery at ${BATTERY_PERCENT}%. Initiating coordinated shutdown."

    # STEP 1: Shutdown App VMs (docker1, docker2)
    echo "Stopping Application VMs..."
    qm shutdown $VM_APP_1 --timeout $TIMEOUT_APP
    qm shutdown $VM_APP_2 --timeout $TIMEOUT_APP

    # STEP 2: Wait for App VMs to finish (optional but recommended)
    # We wait a bit to ensure they stop connecting to the DB before we kill the DB
    echo "Waiting for App VMs to stop gracefully..."
    sleep 30 

    # STEP 3: Shutdown DB VM (docker3)
    echo "Stopping Database VM (PostgreSQL)..."
    qm shutdown $VM_DB --timeout $TIMEOUT_DB

    # STEP 4: Wait for DB VM to finish
    echo "Waiting for Database VM to stop gracefully..."
    sleep 30

    # STEP 5: Shutdown the Proxmox Host itself
    echo "Shutting down Proxmox Host..."
    shutdown -h now
else
    echo "Battery level is safe. Continuing normal operation."
fi
How This Works
qm shutdown <VM_ID>: This command tells the Proxmox host to send an ACPI "Power Button" signal to the VM.
Inside the VM: The OS (Linux) receives this signal. If qemu-guest-agent is running, the OS handles the shutdown sequence:
It stops Docker (systemctl stop docker or similar).
Docker sends SIGTERM to the containers.
PostgreSQL receives the signal, finishes current transactions, and closes cleanly.
Order Matters: By shutting down the App VMs before the DB VM, you ensure that no application tries to write to the database while the database is shutting down.
Alternative: If Guest Agent is NOT Installed
If you cannot install the Guest Agent in the VMs, qm shutdown will default to a hard power-off (equivalent to pulling the plug), which risks database corruption.

In that case, you must use SSH from the host to the VMs to run the shutdown command remotely:

bash

Copy
# Example inside the host script
ssh -o StrictHostKeyChecking=no user@192.168.1.101 "systemctl stop docker && shutdown -h now"
Note: This requires passwordless SSH keys set up between the host and the VMs.

Recommendation: Stick with qm shutdown + Guest Agent. It is the native, most robust way to handle this in Proxmox without managing SSH keys or network dependencies during a power failure.






Qwen VL 235B
I can do the guest agent thing - but I have containers running on proxmox3 as well and 1 and 2. I need to shut them down before shutting down postgresql. I'd like a more fine-grained way to control this. qm shutdown is not fine-grained.
To achieve fine-grained control where you stop specific Docker containers before shutting down the VMs, you should move the shutdown logic inside the VMs rather than relying solely on the Proxmox host to orchestrate the container stops.

The most robust pattern is a Two-Stage Signal:

Host: Detects low battery -> Sends a "Pre-Shutdown" signal (e.g., a file or a specific command) to all VMs.
VMs: Receive the signal -> Run a custom script to stop containers in the correct order -> Then shut down the VM itself.
Here is the best architecture for your setup:

The Strategy: "Signal File" + "Local Orchestration"
Since the network might drop during power loss, relying on the host to SSH into VMs to stop containers is risky. Instead, have the host drop a "flag file" on a shared location (like a small NFS mount or a specific directory if the VMs have direct access to the host's disk via 9p/virtio-fs) OR, more simply, have the host send a custom command via qm guest exec (if Guest Agent is installed) to trigger the local script.

Recommended Approach: qm guest exec to trigger a local script.

This allows the host to say "Run my custom shutdown script now" without needing to know the container names inside the VM.

Step 1: Create a "Smart Shutdown" Script inside each VM
Create a script on every VM (docker1, docker2, docker3) that handles the specific logic for that node.

Location: /usr/local/bin/smart-shutdown.sh (inside the VM) Permissions: chmod +x /usr/local/bin/smart-shutdown.sh

For App VMs (proxmox1, proxmox2):

bash

Copy
#!/bin/bash
LOG="/var/log/ups-shutdown.log"
echo "$(date): Received shutdown signal. Stopping App containers..." >> $LOG

# 1. Stop specific app containers (replace with your actual names)
docker stop app-container-1 app-container-2

# 2. Verify they are stopped
sleep 5

# 3. Shutdown the VM itself
shutdown -h now
For DB VM (proxmox3):

bash

Copy
#!/bin/bash
LOG="/var/log/ups-shutdown.log"
echo "$(date): Received shutdown signal. Stopping PostgreSQL first..." >> $LOG

# 1. Stop the PostgreSQL container specifically
# Ensure this container has a graceful stop timeout configured in docker-compose or run
docker stop postgres-db-container --time 60

# 2. Stop any other local containers
docker stop other-local-container

# 3. Shutdown the VM
shutdown -h now
Step 2: Create the "Orchestrator" Script on the Proxmox Host
This script runs on the physical host. It detects the battery level and sends the trigger command to the VMs in the correct order.

Location: /usr/local/bin/ups-orchestrator.sh (on Proxmox Host)

bash

Copy
#!/bin/bash

# Configuration
BATTERY_THRESHOLD=50
VM_APP_IDS=("101" "102") # VM IDs for docker1, docker2
VM_DB_ID="103"            # VM ID for docker3
SHUTDOWN_SCRIPT_CMD="/usr/local/bin/smart-shutdown.sh" # Path inside the VM

# Check Battery
BATTERY_PERCENT=$(upower -i /org/freedesktop/UPower/devices/battery_BAT0 2>/dev/null | grep percentage | awk '{print $2}' | tr -d '%')
if [ -z "$BATTERY_PERCENT" ]; then
    # Fallback check
    if [ -f /sys/class/power_supply/BAT0/capacity ]; then
        BATTERY_PERCENT=$(cat /sys/class/power_supply/BAT0/capacity)
    fi
fi

echo "Battery: ${BATTERY_PERCENT}%"

if [ -n "$BATTERY_PERCENT" ] && [ "$BATTERY_PERCENT" -le "$BATTERY_THRESHOLD" ]; then
    echo "CRITICAL: Initiating fine-grained shutdown sequence."

    # PHASE 1: Trigger App VMs (101, 102)
    echo "Sending signal to App VMs..."
    for VM_ID in "${VM_APP_IDS[@]}"; do
        # Use qm guest exec to run the script inside the VM
        # --user "root" might be needed depending on your agent config
        qm guest exec $VM_ID -- /bin/bash -c "$SHUTDOWN_SCRIPT_CMD" &
    done

    # Wait a moment for the script to start stopping containers
    # This ensures apps stop connecting to DB before DB stops
    echo "Waiting for App VMs to stop containers..."
    sleep 30

    # PHASE 2: Trigger DB VM (103)
    echo "Sending signal to DB VM..."
    qm guest exec $VM_DB_ID -- /bin/bash -c "$SHUTDOWN_SCRIPT_CMD" &

    # PHASE 3: Wait for DB to finish
    echo "Waiting for DB VM to stop containers..."
    sleep 30

    # PHASE 4: Shutdown Host
    echo "All systems stopped. Shutting down Proxmox Host."
    shutdown -h now
else
    echo "Battery safe."
fi
Why this is better:
Fine-Grained Control: The logic for which containers to stop and in what order lives inside the VM where the containers actually run. You can easily edit /usr/local/bin/smart-shutdown.sh on proxmox3 to stop postgres before redis without touching the host script.
Guest Agent Reliance: It uses qm guest exec, which requires the Guest Agent. This is much safer than SSH because it doesn't rely on the network stack being fully up (it uses the QEMU channel).
Order Enforcement: The host script explicitly waits (sleep 30) between triggering the App VMs and the DB VM. This guarantees the apps stop writing before the DB stops accepting connections.
No Network Dependency for Logic: The host doesn't need to know the container names. It just says "Run the script." The script inside the VM knows the container names.
How to set this up:
Install Guest Agent in all 3 VMs (as discussed previously).
Create the smart-shutdown.sh script inside each VM with the specific container names for that node.
Create the ups-orchestrator.sh on the Proxmox host.
Test: Manually run qm guest exec <VM_ID> -- /bin/bash -c "/usr/local/bin/smart-shutdown.sh" to verify it works before relying on the battery trigger.
This gives you the "orchestration" of the host (timing) combined with the "granularity" of the VMs (container management).
