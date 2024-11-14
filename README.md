![Reigen tackling bash monitoring script](https://static.myfigurecollection.net/upload/pictures/2024/05/24/4007922.gif)

# **Reigen tackling bash monitoring script**



## 1. awk - A Text Processing and Pattern Matching Tool

### The awk command processes lines of text based on patterns and fields. It's especially useful for parsing structured text like logs or CSV files.

`echo "CPU Usage: 25%"`  
`echo "CPU Usage: 25%" | awk '{print $3}'`


## 2. sed - A Stream Editor for Editing Text on the Fly

### sed allows you to perform substitutions, deletions, and other text transformations.

Basic Syntax: `sed 's/[pattern]/[replacement]/' [filename]`

`echo "CPU Load High" | sed 's/High/Low/'`




## 3. Grep - Searches for lines containing a specific pattern.

Basic Syntax: grep [pattern] [filename]

`grep "error" /var/log/syslog`  
`grep "root" /var/log/syslog`  


## 4. Combining

### Processes CPU Monitoring : 

`ps aux`  
`ps aux | grep -v "grep" | grep "firefox"`  
`ps aux | grep -v "grep" | grep "firefox" | awk '{print $1, $2, $3, $11}'`  

 Explanation:

`ps aux` Lists all processes.  
`grep -v "grep"` Excludes any line with the word "grep" to avoid false matches.  
`grep "cfirefox"` Filters to show only processes containing "firefox".  
`awk '{print $1, $2, $3, $11}'` Extracts user ($1), PID ($2), CPU usage ($3), and command ($11).  


 Why Use Each Command?

`grep` helps to select specific processes (like "firefox").  
`awk` is then used to focus on specific details, like the CPU usage column.  

What's really important : 

`ps aux | grep -v "grep" | grep "firefox" | awk '{sum += $3} END {print "Total CPU% used by Firefox:", sum"%"}'`  

`ps aux` Lists all running processes.  
`grep -v "grep"` Excludes the grep process itself from the results.  
`grep "firefox"` Filters only the processes related to firefox.  
`awk '{sum += $3} END {print "Total CPU% used by firefox:", sum"%"}'` :   
`sum += $3` Adds the CPU usage ($3) of each Firefox process to the sum variable.  
`END {print "Total CPU% used by firefox:", sum"%"}'` After processing all lines, prints the total CPU usage of all firefox processes with a % symbol appended.  

### Memory Monitoring with grep and awk

`ps aux | grep "firefox" | awk '{sum += $4} END {print "Total Memory Usage by firefox:", sum, "%"}'`

Explanation:

`ps aux` Lists all running processes.  
`grep "firefox"` Filters to show only lines with "apache" processes.  
`awk '{sum += $4} END {print "Total Memory Usage by Apache:", sum, "%"}'` Adds up memory usage ($4 column) for each firefox process and prints the total.  

### Disk Monitoring with grep awk and sed 

`df`  
`df -m`  
`df -h` 

`df -h | grep "/dev/sda2" | awk '{print $1, $3 " used,", $4 " available"}' | sed 's|/dev/sda2|my-personnal-disk|'`

replacing /dev/sda2 by "my-personnal-disk"

`df -h | grep "/dev/sda2" | awk '{print $1, $3 " used,", $4 " available"}' | sed -n 's|/dev/\(sda2\)|\1|p'`

Breakdown of how sed works here :

`/dev/` Matches the literal /dev/ at the beginning of the device name.

`\(...\)` Defines a capturing group around sda2. Anything inside the parentheses is "captured" for later reference.

`sda2` The specific part we want to capture (or display alone).

`\1` Refers to the text captured by the first capturing group (in this case, sda2).

When we replace /dev/sda2 with \1, it effectively outputs just sda2.

### Network Monitoring

`ifconfig`

`ifconfig enp0s3 | grep "RX packets" | awk '{print "Received Packets:", $3}'`

Explanation:
`ifconfig eth0` Displays network configuration for the eth0 interface.  
`grep "RX packets"` Filters to show only the line with received packets.  
`awk '{print "Received Packets:", $3}'` Extracts and labels the packet count from the line.  


`vnstat -tr 2 | grep "rx\|tx" | awk '{print $1, $2, $5}'`


## Final exemple and thing you can do


### 1.Looping.

``` bash
while true; do
  # Fetch CPU and memory usage details
  CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
  MEM_USAGE=$(free | awk '/Mem/ {printf "%.2f", $3/$2 * 100.0}')

  # Print CPU and Memory usage with cursor control
  echo -ne "CPU Usage: $CPU_USAGE%   Memory Usage: $MEM_USAGE%  \r"
  
  # Wait for 2 seconds before refreshing
  sleep 2
done
```

Echo explanation : 

`-n` Prevents echo from adding a new line.  
`-e` Enables interpretation of escape characters.  
`\r:` Returns the cursor to the beginning of the line, allowing the new output to overwrite the old output.  

Result: 

Only the CPU and memory usage percentages are refreshed every 2 seconds, maintaining a clean single-line display.

### 2. Ascii board + partial clearing

``` bash
# Define a function to set up the board layout
draw_board() {
  clear
  echo "==================================="
  echo "|           CPU MONITOR           |"
  echo "==================================="
  echo "|  CPU Usage:                   % |"
  echo "==================================="
}

# Draw the board once
draw_board

# Infinite loop to update only the CPU percentage
while true; do
  # Fetch the current CPU usage
  CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')

  # Move the cursor to the specific position (row 4, column 20) and update CPU usage
  tput cup 3 18
  echo -ne "$CPU_USAGE"

  # Wait for 2 seconds before updating
  sleep 2
done
```

Explanation of the Code

`draw_board Function` Sets up a simple ASCII board that will not change. This board layout is static and will not be redrawn, keeping the rest of the screen stable.

`clear` Clears the terminal screen initially.
The board is drawn with fixed = and | symbols to create an ASCII "box" around the CPU usage information.
Cursor Control with tput:

`tput cup 3 18` Moves the cursor to a specific location on the screen, where the CPU percentage will be displayed.
3 refers to the row (line 4 of the board, counting from 0).
18 refers to the column (after "CPU Usage:" text).
By positioning the cursor here, we can overwrite just the percentage value, without affecting the rest of the ASCII board.

Updating CPU Usage:

`echo -ne "$CPU_USAGE"` Prints the CPU usage percentage at the cursor’s position without moving to a new line.  
`sleep 2` Pauses for 2 seconds before the next update.

Result:

Only the percentage part in the CPU usage row will be refreshed every 2 seconds, keeping the rest of the board untouched.

### 3. ANSI color code

```bash
# Define color variables
WHITE="\033[1;37m"
GREEN="\033[1;32m"
YELLOW="\033[1;33m"
RED="\033[1;31m"
RESET="\033[0m"

# Define a function to set up the board layout
draw_board() {
  clear
  echo "==================================="
  echo "|           CPU MONITOR          |"
  echo "==================================="
  echo "|  CPU Usage:                   % |"
  echo "==================================="
}

# Draw the board once
draw_board

# Infinite loop to update only the CPU percentage
while true; do
  # Fetch the current CPU usage
  CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')

  # Move the cursor to the specific position (row 4, column 20) and update CPU usage with a color
  tput cup 3 18
  echo -ne "${GREEN}$CPU_USAGE%${RESET}"  # Using GREEN color for percentage

  # Wait for 2 seconds before updating
  sleep 2
done
```

Explanation of the Code with Colors

Color Variables:

WHITE, GREEN, YELLOW, RED: These variables store ANSI escape codes for different colors:
WHITE: Bold white.
GREEN: Bold green.
YELLOW: Bold yellow.
RED: Bold red.
RESET: Resets text color back to the default after printing.

Example Usage: ${GREEN}, ${YELLOW}, ${RED}, etc., can be used to apply colors to specific parts of the output.

Using Colors in the CPU Usage Output:

`${GREEN}$CPU_USAGE%${RESET}` Prints the CPU usage percentage in green. The `${RESET}` variable ensures the color does not affect any text printed afterward.
This is easy to change—simply replace `${GREEN}` with `${YELLOW}`, `${WHITE}`, etc., to switch colors as needed.

Result:

Only the CPU usage percentage is refreshed in green every 2 seconds, with the rest of the board layout remaining static. You can swap ${GREEN} to another color variable for quick adjustments.



Where you want to search the data you want for your monitoring script :

`top`  
`top -bn1` : non interactive  

Usage: top  
What it does: Displays real-time system resource usage, showing CPU, memory, and a list of running processes. You could ask the audience to try identifying the most resource-hungry process.

`df -h`

Usage: df -h  
What it does: Shows disk usage of file systems in a human-readable format (sizes in MB/GB). Great for checking free disk space.

`du -sh *`

Usage: du -sh *  
What it does: Provides the size of each file/folder in the current directory. Helps with finding large files taking up space.

`free -m`

Usage: free -m  
What it does: Displays memory usage in megabytes. Useful for checking RAM and swap space usage.

`iostat`

Usage: iostat  
What it does: Shows CPU and device I/O statistics. This can help you find potential bottlenecks in disk read/write speeds. (Install with sudo apt install sysstat if needed.)

`netstat -tuln`

Usage: netstat -tuln  
What it does: Lists active listening ports, useful for identifying network services.

`ps aux | grep <service_name>`

Usage: ps aux | grep firefox  
What it does: Filters processes to see if a specific service is running. Replace <service_name> with any process name.

`uptime`

Usage: uptime  
What it does: Shows how long the system has been running, along with average load. Great for checking stability.

`vmstat 1 5`

Usage: vmstat 1 5  
What it does: Monitors system performance by showing stats every second for five times. Good for checking real-time memory, swap, and CPU.

## **I hope It did help u guys.**

![Reigen leaving](https://i.pinimg.com/originals/7d/86/8e/7d868e530bd4ffac7bed8dca9cbb8d63.gif)

# **Reigen is now leaving**
