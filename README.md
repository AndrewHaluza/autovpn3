1) `sudo apt install openvpn`
2) `gedit autovpn3.sh`
3) Paste:
```
#!/bin/bash
 
# autovpn3, coded by MiAl, 
# you can leave a bug report on the page: https://miloserdov.org/?p=5858
# сообщить об ошибке на русском вы можете на странице: https://HackWare.ru/?p=15429
 
# you can change these parameters:
country='' # empty for any or JP, KR, US, TH, etc.
useSavedVPNlist=0 # set to 1 if you don't want to download VPN list every time you restart this script, otherwise set to 0
useFirstServer=0 # set the value to 0 to choose a random VPN server, otherwise set to 1 (maybe the first one has higher score)
vpnList='/tmp/vpns.tmp'
proxy=0 # replace with 1 if you want to connect to VPN server through a proxy
proxyIP=''
proxyPort=8080
proxyType='socks' # socks or http
 
# don't change this:
counter=0
VPNproxyString=''
cURLproxyString=''
 
if [ $proxy -eq 1 ];then
    echo 'We will use a proxy'
    if [ -z "$proxyIP" ]; then
        echo "To use a proxy, you must specify the proxy's IP address and port (hardcoded in the source code)."
        exit
    else
        if [ "$proxyType" == "socks" ];then
            VPNproxyString=" --socks-proxy $proxyIP $proxyPort "
            cURLproxyString=" --proxy socks5h://$proxyIP:$proxyPort "
        elif [ "$proxyType" == "http" ];then
            VPNproxyString=" --http-proxy $proxyIP $proxyPort "
            cURLproxyString=" --proxy http://$proxyIP:$proxyPort "
        else
            echo 'Unsupported proxy type.'
            exit   
        fi 
    fi
fi
 
if [ $useSavedVPNlist -eq 0 ];then
    echo 'Getting the VPN list'
    curl -s $cURLproxyString https://www.vpngate.net/api/iphone/ > $vpnList
elif [ ! -s $vpnList ];then
    echo 'Getting the VPN list'
    curl -s $cURLproxyString https://www.vpngate.net/api/iphone/ > $vpnList  
else
    echo 'Using existing VPN list'
fi
 
while read -r line ; do
    array[$counter]="$line"
    counter=$counter+1
done < <(grep -E ",$country" $vpnList)
 
CreateVPNConfig () {
    if [ -z "${array[0]}" ]; then
        echo 'No VPN servers found from the selected country.'
        exit
    fi
 
    size=${#array[@]}
 
    if [ $useFirstServer -eq 1 ]; then
        index=0
        echo ${array[$index]} | awk -F "," '{ print $15 }' | base64 -d > /tmp/openvpn3
    else       
        index=$(($RANDOM % $size))
        echo ${array[$index]} | awk -F "," '{ print $15 }' | base64 -d > /tmp/openvpn3
    fi
 
    echo 'Choosing a VPN server:'
    echo "Found VPN servers: $((size+1))"
    echo "Selected: $index"
    echo "Country: `echo ${array[$index]} | awk -F "," '{ print $6 }'`"   
}
 
while true
    do
        CreateVPNConfig
        echo 'Trying to start OpenVPN client'
        sudo openvpn --config /tmp/openvpn3 $VPNproxyString
        read -p "Try another VPN server? (Y/N): " confirm && [[ $confirm == [yY] || $confirm == [yY][eE][sS] ]] || exit
    done
```

4) chmod +x autovpn3.sh
5) sudo ./autovpn3.sh
