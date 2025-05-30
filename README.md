Виконала самостійно Слобожан Софія РПЗ-23А
# WORK-CASE-7
# Task Scheduling in Operating Systems

## 1. Functions of a Task Scheduler

A task scheduler in any operating system (OS) is responsible for managing the execution of processes and tasks. The main functions of a task scheduler include:

- **Task Prioritization**: Assigning priority levels to tasks to ensure that critical processes receive more CPU time and resources.
- **Time Slicing**: Dividing CPU time among multiple tasks to allow for multitasking, enabling the system to run several processes concurrently.
- **Context Switching**: Saving the state of a currently running task and loading the state of the next task to be executed, allowing for efficient multitasking.
- **Resource Management**: Allocating system resources such as CPU, memory, and I/O devices to tasks as needed, ensuring optimal performance.
- **Task Creation and Termination**: Managing the lifecycle of tasks, including their creation, execution, and termination.
- **Scheduling Algorithms**: Implementing various algorithms (e.g., Round Robin, First-Come-First-Served, Shortest Job First) to determine the order of task execution.

### Comparison of Task Scheduling in Windows and Linux

- **Windows**:
  - Utilizes a preemptive multitasking model with a priority-based scheduling algorithm.
  - The Windows Task Scheduler provides a graphical interface for users to schedule tasks based on time or events, such as system startup or user logon.
  - Supports a variety of triggers and conditions for task execution, making it user-friendly for non-technical users.

- **Linux**:
  - Also employs a preemptive multitasking model but offers more flexibility with various scheduling algorithms (e.g., Completely Fair Scheduler, Real-Time Scheduler).
  - The `cron` daemon is a widely used tool for scheduling tasks based on time intervals, allowing users to automate repetitive tasks through command-line interfaces.
  - Linux provides more granular control over task scheduling, which can be advantageous for advanced users and system administrators.

## 2. Working with Cron in Linux

### Principles of Cron

`cron` is a time-based job scheduler in Unix-like operating systems. It allows users to schedule jobs (commands or scripts) to run at specified intervals. The main principles include:

- **Cron Jobs**: Defined in a crontab file, which specifies the timing and command to execute.
- **Time Format**: The timing is specified using five fields: minute, hour, day of month, month, and day of week.
- **User -Specific Crontabs**: Each user can have their own crontab file, allowing for personalized scheduling.

### Configuration of Cron

To configure `cron`, follow these steps:

1. **Open the Terminal**: Access the command line interface.
2. **Edit the Crontab File**: Use the command `crontab -e` to open the crontab file in an editor.
3. **Add Cron Job Entries**: Specify the desired cron job in the format:
   ```
   * * * * * command_to_execute
   ```
   where each asterisk represents a time field.

### Alternatives to Cron

- **Anacron**: Similar to `cron`, but designed for systems that may not be running continuously. It ensures that scheduled tasks are executed even if the system was off during the scheduled time.
- **Systemd Timers**: A more modern alternative that integrates with the systemd init system, allowing for more complex scheduling and dependencies. It can handle tasks based on time and events, providing greater flexibility than traditional cron jobs.

# Backup Script and Scheduling Report

## 1. Creation of Backup Script

To automate the backup process, a bash script was created. The following steps outline the creation and configuration of the backup script:

![image](https://github.com/user-attachments/assets/77ee8799-548e-4202-9c76-240a76eeff65)

### Step 1: Create the Script Directory

First, a directory for scripts was created:

```bash
mkdir -p "$HOME/scripts"
```

### Step 2: Create the Backup Script

Next, the backup script was created using a here-document to insert the script content:

```bash
cat << 'EOF' > "$HOME/scripts/backup.sh"
#!/bin/bash

BACKUP_DIR="$HOME/backups"
mkdir -p "$BACKUP_DIR"

# Archive the Documents folder
tar -czf "$BACKUP_DIR/home_documents_$(date +%F_%H-%M).tar.gz" "$HOME/Documents" 2>/dev/null

# Delete archives older than 7 days
find "$BACKUP_DIR" -type f -name "*.tar.gz" -mtime +7 -delete
EOF
```

### Step 3: Make the Script Executable

To ensure the script can be executed, the following command was run:

```bash
chmod +x "$HOME/scripts/backup.sh"
```

### Step 4: Manual Execution for Testing

The script was executed manually to verify its functionality:

```bash
echo "Running the script manually for verification..."
"$HOME/scripts/backup.sh"
```

### Step 5: List Backups

The contents of the backup directory were listed to confirm the creation of the backup:

```bash
echo "Archives in the backups folder:"
ls -lh "$HOME/backups"
```

## 2. Scheduling Tasks with Crontab

![image](https://github.com/user-attachments/assets/43c5f6f5-7371-44a7-8107-6c969715294b)
![image](https://github.com/user-attachments/assets/b282e9fd-718f-4b04-a472-dc8867ded5be)


To automate the execution of the backup script, several cron jobs were scheduled using `crontab`.

### Step 1: Function to Add Cron Jobs

A function was defined to add cron jobs without duplicating existing entries:

```bash
add_cron_job() {
  (crontab -l 2>/dev/null | grep -v -F "$1"; echo "$1") | crontab -
}
```

### Step 2: Define the Script Path

The path to the backup script was defined:

```bash
SCRIPT_PATH="$HOME/scripts/backup.sh"
```

### Step 3: Add Cron Job Entries

The following cron jobs were added to schedule the backup script:

- **Daily at 08:00 AM**:
  ```bash
  add_cron_job "0 8 * * * $SCRIPT_PATH"
  ```

- **Twice a Day at 08:00 AM and 06:30 PM**:
  ```bash
  add_cron_job "0 8,18 * * * $SCRIPT_PATH"
  ```

- **Weekdays (Monday to Friday) from 08:00 AM to 06:00 PM every hour**:
  ```bash
  add_cron_job "0 8-18 * * 1-5 $SCRIPT_PATH"
  ```

- **Once a Year on January 1st at 00:00**:
  ```bash
  add_cron_job "0 0 1 1 * $SCRIPT_PATH"
  ```

- **Once a Month on the 1st at 02:00**:
  ```bash
  add_cron_job "0 2 1 * * $SCRIPT_PATH"
  ```

- **Once a Day at 03:00**:
  ```bash
  add_cron_job "0 3 * * * $SCRIPT_PATH"
  ```

- **Every Hour**:
  ```bash
  add_cron_job "0 * * * * $SCRIPT_PATH"
  ```

- **At System Startup**:
  ```bash
  add_cron_job "@reboot $SCRIPT_PATH"
  ```

### Step 4: Display Updated Crontab

The updated crontab was displayed to confirm the scheduled jobs:

```bash
echo "Updated crontab:"
crontab -l
```

## 3. Alternative: Systemd Timer
![image](https://github.com/user-attachments/assets/4052e2f5-87cf-4e4b-baee-ec85c936193e)

An alternative to using `cron` for scheduling tasks is to use `systemd` timers. The following steps outline the creation and activation of a systemd timer for the backup script.

### Step 1: Create the Service File

A service file was created for the backup script:

```bash
SERVICE_PATH="/etc/systemd/system/backup.service"
sudo tee "$SERVICE_PATH" > /dev/null <<EOF
[Unit]
Description=Backup Documents (systemd service)

[Service]
Type=oneshot
ExecStart=$SCRIPT_PATH
EOF
```

### Step 2: Create the Timer File

Next, a timer file was created:

```bash
TIMER_PATH="/etc/systemd/system/backup.timer"
sudo tee "$TIMER_PATH" > /dev/null <<EOF
[Unit]
Description=Daily Backup at 08:00

[Timer]
OnCalendar=*-*-* 08:00:00
Persistent=true

[Install]
WantedBy=timers.target
EOF
```

### Step 3: Activate the Timer

To activate the timer, the following commands were executed:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
sudo systemctl start backup.service
```

### Step 4: List Backups After Timer Execution

The contents of the backup directory were listed again to confirm the creation of the backup after the timer was activated:

```bash
echo "Archives after running the systemd service:"
ls -lh "$HOME/backups"
```

### Step 5: Check the Status of the Systemd Timer

The status of the systemd timer was checked to ensure it is running:

```bash
echo -e "\nStatus of the systemd timer:"
systemctl list-timers | grep backup
```

## Conclusion

This report outlines the creation of a backup script, its scheduling using `crontab`, and the implementation of an alternative scheduling method using `systemd` timers. Both methods provide effective solutions for automating the backup process, ensuring that important data is regularly backed up and maintained. The successful execution of the backup script and the verification of the created archives demonstrate the effectiveness of the implemented solutions.
