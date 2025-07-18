```bash
# Initialize a counter variable
i=1

# Loop through each unique IP address from the log file
for ip in $(grep "Amazonbot" /var/log/apache2/access.log | awk '{print $1}' | uniq); do
  # Print the loop index (counter) and the IP being processed
  echo "Adding rule #$i: Blocking IP $ip"
  
  # Add the iptables rule to drop packets from the source IP
  sudo iptables -A INPUT -s $ip -j DROP
  
  # Increment the counter for the next loop
  ((i++))
done

echo "Finished. Total of $((i-1)) rules added."

```