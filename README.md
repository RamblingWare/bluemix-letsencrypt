# bluemix-letsencrypt

A script for configuring [Let's Encrypt](https://letsencrypt.org) SSL certificates for CloudFoundry apps on IBM Bluemix.

## Prerequisites

Assuming you must have the custom domains created, DNS configured, and set your target of choice.

 * [Bluemix CLI](https://clis.ng.bluemix.net/ui/home.html)
 * [Python 3.6+](https://www.python.org/downloads/)

## Usage

 1. Clone:
    ```
    git clone https://github.com/RamblingWare/bluemix-letsencrypt`
    cd bluemix-letsencrypt
    ```
 1. Install dependencies
    ```
    pip install -r requirements.txt
    ```
 1. Edit domains.yml with your email and custom domain names
 1. Start: `python setup-app.py`
    1. Map route needed for Let's Encrypt to verify you own the domain
       ```
       bx cf map-route letsencrypt www.ramblingware.com --path .well-known/*
       bx cf map-route letsencrypt www.ramblingware.com --path .well-known/acme-challenge/
       ```
    1. Let the app finish verification. Check logs with `cf logs letsencrypt --recent`
 1. Download the resulting certificate files
    ```
    bx cf ssh letsencrypt -c 'cat ~/app/conf/live/www.ramblingware.com/cert.pem' > cert.pem
    bx cf ssh letsencrypt -c 'cat ~/app/conf/live/www.ramblingware.com/chain.pem' > chain.pem
    bx cf ssh letsencrypt -c 'cat ~/app/conf/live/www.ramblingware.com/fullchain.pem' > fullchain.pem
    bx cf ssh letsencrypt -c 'cat ~/app/conf/live/www.ramblingware.com/privkey.pem' > privkey.pem
    ```
 1. Upload certificate files to Bluemix for your custom domain
    1. `bx app domain-cert-remove www.ramblingware.com`
    1. `bx app domain-cert-add www.ramblingware.com -c cert.pem -k privkey.pem -i chain.pem`
    1. `bx app domain-cert www.ramblingware.com`

## Note

> During testing, please set `staging` to `true` in order to keep load off the production Let's Encrypt environment and reduce the chance of hitting their [rate limits](https://letsencrypt.org/docs/staging-environment/).

> If you're using HSTS or redirecting traffic from http to https at the network level (DNS), you'll have to disable it temporarily so that letsencrypt can reach your domain with http to do the acme challenge.

> Sometimes bluemix returns `500` errors for modifying certificates on a domain. Just try again a few times until the request goes through.
