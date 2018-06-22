# nsupdate-client


A simple client to update DNS record from a remote client to a DNS server using nsupdate protocol, like No-IP or DynDNS for your own domain names.

Tested on Linux.

## Requirements

* A DNS Server configured to accept updates (see below)
* Firewall with port 53/UDP open on the server
* dnsutils package on your server & clients

## Configure your DNS server

Generate a public/private key pair :

    tsig-keygen -a HMAC-SHA512 www.domain.com >  ddns-key.www.domain.fr.key

Copy ddns-key.www.domain.fr.key to you host.

Configure your DNS zone like this:

    zone "domain.com" {
      type master;
      file "/etc/bind/domain.com.hosts";
      update-policy {
            grant ddns-key.www.domain.fr name www.domain.fr ANY;
        };
    };

    key "www.domain.com" {
      algorithm hmac-md5;
      secret "secret value on file ddns-key.www.domain.fr.key";
    };

Restart your DNS Server

    systemctl restart bind9


## Usage on your client

Clone the project 

    git clone git@github.com:cweiland/nsupdate-client.git

And then choose the command that fit your needs:

    With static IP
    ./nsupdate-client -k ddns-key.www.domain.fr.key -r www.domain.fr -l 600 -t A -d 192.168.1.100

    With internal dynamic IP (where ethX is your interface name, like eth0, en1, ppp2...)
    ./nsupdate-client -k ddns-key.www.domain.fr.key -r www.domain.fr -l 600 -t A -d if_ethX

    With external dynamic IP (update only when external ip changes)
    ./nsupdate-client -k ddns-key.www.domain.fr.key -r www.domain.fr -l 600 -t A -d external

    With CNAME
    ./nsupdate-client -k ddns-key.www.domain.fr.key -r www.domain.fr -l 600 -t CNAME domain.fr


