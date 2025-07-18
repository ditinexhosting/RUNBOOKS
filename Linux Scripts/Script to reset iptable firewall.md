# Set the default policies for each of the built-in chains to ACCEPT.
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT

# Flush all rules from all chains.
sudo iptables -F

# Delete all non-default user-defined chains.
sudo iptables -X

# Flush all rules in the 'nat' and 'mangle' tables.
sudo iptables -t nat -F
sudo iptables -t mangle -F

# Delete all non-default user-defined chains in the 'nat' and 'mangle' tables.
sudo iptables -t nat -X
sudo iptables -t mangle -X
