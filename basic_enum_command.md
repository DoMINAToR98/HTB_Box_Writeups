# Basic enumeration commands

## nmap

### All ports, with services

```
nmap -sV -p- -T5 {HOST_IP}
```

### All ports, no services

```
nmap -p- -T5 {HOST_IP}
```

### 1000 most frequent ports, with services

```
nmap {HOST_IP}
```

## gobuster

```
gobuster -u {your site} -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

## Misc. commands

### To save output on a file, but also output into stdout

```
command | tee {YOUR_FILE}
```
