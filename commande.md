## Desactiver 2FA sur TrueNAS :
midclt call auth.twofactor.update '{"enabled": false}'
systemctl restart middlewared
> pour verifier :
  midclt call auth.twofactor.config