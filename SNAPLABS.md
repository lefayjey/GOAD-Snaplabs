Installation:

1. Create boxes on Snaplabs as follow:
# windows server 2019
name => "DC01",  :ip => "192.168.56.10" | kingslanding.sevenkingdoms.local
# windows server 2019
name => "DC02",  :ip => "192.168.56.11" | winterfell.north.sevenkingdoms.local
# windows server 2016
name => "DC03",  :ip => "192.168.56.12" | meereen.essos.local
# windows server 2019
name => "SRV02", :ip => "192.168.56.22" | castelblack.north.sevenkingdoms.local
# windows server 2016
name => "SRV03", :ip => "192.168.56.23" | braavos.essos.local
# Ansible - ubuntu 20.04
name => "ansible", :ip => "192.168.56.19"
# Kali - Kali Linux
name => "kali", :ip => "192.168.56.5"
# ELK - ubuntu 18.04
name => "elk", :ip => "192.168.56.50"

Memory:
Windows DCs => Memory = 1000, CPUs = 2, Disk Size 30GB
Windows SRV02 => Memory = 4000, CPUs = 2, Disk Size 30GB
Windows SRV03 => Memory = 2000, CPUs = 2, Disk Size 30GB
Ansible => Memory = 2000, CPUs = 2, Disk Size 30GB
ELK => Memory = 4000, CPUs = 2, Disk Size 60GB
Kali => Memory = 4000, CPUs = 2, Disk Size 64GB

2. Create Inventory file and copy SSH key under Settings in the Snaplabs' range:
# ad/GOAD/providers/snaplabs/inventory
--------------------------------------
Replace ansible creds and elk key:
##   ansible_user=snaplabs
##   ansible_password=5n4pl4b5!!
## ansible_ssh_user=ubuntu
## ansible_ssh_private_key_file=../ad/GOAD/providers/snaplabs/goad.pem

Change Ethernet configuration:
# two_adapters=no
# nat_adapter=Ethernet 2
# domain_adapter=Ethernet 2

3. Modify the ad-trusts.yml file:
# ansible/ad-trusts.yml
--------------------------------
Comment the following lines
#  - { role: 'settings/disable_nat_adapter' , tags: 'disable_nat_adapter'}
#  - { role: 'dns_conditional_forwarder', tags: 'dns_conditional_forwarder' }
#  - { role: 'settings/enable_nat_adapter', tags: 'enable_nat_adapter'}

4. Execute manually on all servers:
wget https://raw.githubusercontent.com/lefayjey/GOAD/main/vagrant/ConfigureRemotingForAnsible.ps1 -O ConfigureRemotingForAnsible.ps1
.\ConfigureRemotingForAnsible.ps1 -Verbose
net user snaplabs 5n4pl4b5!! /add; net localgroup administrators snaplabs /add

5. Launch the goad.sh script from the ansible machine
./goad.sh -t install -l GOAD -p snaplabs -m docker

6. Manually Change the DNS suffix on all systems (add the domain) and remove EC2 Certificates from Personal

7. Add 2 DNS entries on DC02 (Fix limitation of LLMNR in AWS)
Add-DnsServerResourceRecordA -Name "bravos" -ZoneName "north.sevenkingdoms.local" -AllowUpdateAny -IPv4Address "192.168.56.5" -TimeToLive 01:00:00
Add-DnsServerResourceRecordA -Name "meren" -ZoneName "north.sevenkingdoms.local" -AllowUpdateAny -IPv4Address "192.168.56.5" -TimeToLive 01:00:00

8. Refresh network adapter after startup
$trigger = New-JobTrigger -AtStartup -RandomDelay 00:00:30
Register-ScheduledJob -Trigger $trigger -ScriptBlock {Restart-NetAdapter -Name "Ethernet 2"} -Name RestartNetAdapter
$principal = New-ScheduledTaskPrincipal -UserID "NT AUTHORITY\SYSTEM" -LogonType ServiceAccount  -RunLevel Highest
Set-ScheduledTask -TaskPath "\Microsoft\Windows\PowerShell\ScheduledJobs" -TaskName "RestartNetAdapter"  -Principal $principal

9. Generate new certs according to new hostname
.\ConfigureRemotingForAnsible.ps1 -Verbose -ForceNewSSLCert 
del C:\ConfigureRemotingForAnsible.ps1
certutil –pulse

10. Clean terminal history
del	C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHistory.txt
del C:\Users\snaplabs\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHistory.txt