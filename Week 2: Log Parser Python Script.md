# Simple log parser - extracts failed login attempts
# Input: wazuh-alerts.log          
# Output: Count of failed logins by IP, timestamp of last attempt

import re

def parse_log(file_path):
    """
    Parse a log file and count failed login attempts by IP address.
    """
    failed_logins = {}
    ip_pattern = re.compile(r'\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b')
    
    with open(file_path, 'r') as f:
        for line in f:
            # Check for failed login indicators
            if "Failed password" in line or "Logon Failure" in line:
                match = ip_pattern.search(line)
                if match:
                    ip = match.group()
                    failed_logins[ip] = failed_logins.get(ip, 0) + 1
    
    return failed_logins

def main():
    results = parse_log("sample_logs.txt")
    
    print(f"Found {len(results)} unique IPs with failed logins")
    print("-" * 40)
    
    # Sort by count (highest first) for prioritisation
    for ip, count in sorted(results.items(), key=lambda x: x[1], reverse=True):
        print(f"{ip:<20} {count:>3} attempts")

if __name__ == "__main__":
    main()
