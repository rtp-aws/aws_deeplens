In registration, you download this cert in one step and then upload it 
back to the device after you connect it to your wifi.

The certificates file is an encrypted zip file.  It was created via this command

```
gpg --output certificates-deeplens_ZF0JFQ1CQTi-gC5zuctrWQ.zip.gpg --encrypt --recipient davisjf@gmail.com certificates-deeplens_ZF0JFQ1CQTi-gC5zuctrWQ.zip
```

To decrypt the file use this command

```
gpg --output certificates-deeplens_ZF0JFQ1CQTi-gC5zuctrWQ.zip --decrypt certificates-deeplens_ZF0JFQ1CQTi-gC5zuctrWQ.zip.gpg
```


