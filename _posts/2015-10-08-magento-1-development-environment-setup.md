There is an advanced Magento development environment setup tool implemented by [davidalger](https://github.com/davidalger/devenv){:target="_blank"}. But I'd like to keep my env simple and basic:P So I decided to make my own.

The Env is a simple LAMP server with some basic automations based on Vagrant and Virtualbox. I chose to set it up with the most popular Vagrant box [ubuntu/trusty64](https://atlas.hashicorp.com/ubuntu/boxes/trusty64){:target="_blank"}. Then, I wrote a simple shell script lamp.sh as a provisoning file to install LAMP and set up the demo site mage.local. If you need another brand new demo site, simply call: ```sh alpha.sh```. This will set up another site: alpha.local. Otherwise, you can recreate the mage.local demo site by calling ```sh mage.local```. 

This setup is basic. Everything is static, but it suffices to boost my local development anyway:P

Check it out: [https://github.com/dujushi/magento-I-development-env](https://github.com/dujushi/magento-I-development-env){:target="_blank"}.