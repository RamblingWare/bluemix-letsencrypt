# bluemix-letsencrypt
A script for configuring [Let's Encrypt](https://letsencrypt.org) SSL certificates for CloudFoundry apps on IBM Bluemix

## Prerequisites

Firstly you must have the custom domains created, DNS configured, and set your target of choice.

 * [CF cli](https://github.com/cloudfoundry/cli/releases)
 * [BX cli](https://clis.ng.bluemix.net/ui/home.html)
 * [Python 2.7](https://www.python.org/downloads/)

## Usage

 1. Clone: `git clone https://github.com/RamblingWare/bluemix-letsencrypt`
 1. `cd bluemix-letsencrypt`
 1. Install dependencies
    * `pip install requests`
    * `pip install pyyaml`
 1. Edit domain.yml with your email and custom domain names
 1. Start: `python setup-app.py`
    1. Map route needed for Let's Encrypt to verify you own the domain
       * `cf map-route letsencrypt www.ramblingware.com --path .well-known/*`
    1. Let the app finish verification.
       * Check logs with `cf logs letsencrypt --recent`
 1. Download the resulting certificate files
    * `cf ssh letsencrypt -c 'cat ~/app/conf/live/www.ramblingware.com/cert.pem' > cert.pem`
    * `cf ssh letsencrypt -c 'cat ~/app/conf/live/www.ramblingware.com/chain.pem' > chain.pem`
    * `cf ssh letsencrypt -c 'cat ~/app/conf/live/www.ramblingware.com/fullchain.pem' > fullchain.pem`
    * `cf ssh letsencrypt -c 'cat ~/app/conf/live/www.ramblingware.com/privkey.pem' > privkey.pem`
 1. Upload certificate files to Bluemix for your custom domain
    1. `bx app domain-cert-remove www.ramblingware.com`
    1. `bx app domain-cert-add www.ramblingware.com -c cert.pem -k privkey.pem -i chain.pem`
    1. `bx app domain-cert www.ramblingware.com`

> Note: During testing, please set `staging` to `true` in order to keep load off the production Let's Encrypt environment and reduce the chance of hitting their [rate limits](https://letsencrypt.org/docs/staging-environment/).
