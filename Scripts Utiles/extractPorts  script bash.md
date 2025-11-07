

```
extractPorts () {
    ports="$(cat $1 | grep -oP '\d{1,5}/open' | awk '{print $1}' FS='/' | xargs | tr ' ' ',')"
    ip_address="$(cat $1 | grep -oP '^Host: .* \(\)' | head -n 1 | awk '{print $2}')"
    echo -e "\n[*] Extracting information...\n" > extractPorts.tmp
    echo -e "\t[*] IP Address: $ip_address" >> extractPorts.tmp
    echo -e "\t[*] Open ports: $ports\n" >> extractPorts.tmp
    echo $ports | tr -d '\n' | xclip -sel clip
    echo -e "[*] Ports copied to clipboard\n" >> extractPorts.tmp
    cat extractPorts.tmp
    rm extractPorts.tmp
}
```