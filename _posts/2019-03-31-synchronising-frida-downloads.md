When conducting mobile application penetration tests, many pentesters or security analysts will come across various tools using Frida [[^1]].

I have made one too many mistake in the past of using the wrong _frida-server_ version that was incompatible with the frida version that was being utilised.

Here is a little short script to synchronise the same Frida versions and has been tailored to download frida-servers specific to iOS and Android pentests. 


***

Copy the following script and save it as a _frida.sh_.


```
#!/bin/sh

# Getting the latest Frida version
tmpfile=tmp.json
curl -s -X GET https://api.github.com/repos/frida/frida/tags -o $tmpfile
LATEST_RELEASE=`cat $tmpfile |grep name | head -1 |  sed 's/\"//g' |  sed 's/\,//g'|  gawk '{split($0,array,": ")} END{print array[2]}'`
rm $tmpfile

# Message
echo Updating frida to $LATEST_RELEASE
echo $LATEST_RELEASE > "Frida Version"

# Downloading frida-servers for iOS and Android
curl -L -O -J https://github.com/frida/frida/releases/download/$LATEST_RELEASE/frida-server-$LATEST_RELEASE-android-x86_64.xz
curl -L -O -J https://github.com/frida/frida/releases/download/$LATEST_RELEASE/frida-server-$LATEST_RELEASE-android-x86.xz
curl -L -O -J https://github.com/frida/frida/releases/download/$LATEST_RELEASE/frida-server-$LATEST_RELEASE-android-arm64.xz
curl -L -O -J https://github.com/frida/frida/releases/download/$LATEST_RELEASE/frida-server-$LATEST_RELEASE-android-arm.xz
curl -L -O -J https://github.com/frida/frida/releases/download/$LATEST_RELEASE/frida-server-$LATEST_RELEASE-ios-arm64.xz
curl -L -O -J https://github.com/frida/frida/releases/download/$LATEST_RELEASE/frida-server-$LATEST_RELEASE-ios-arm.xz

# Unpack, unpack....
for f in *.xz
 do
  # unxz x "$f"
  unxz "$f"
 done
rm *.xz
```

Launching your terminal of choice, type `./frida.sh` at the prompt.

***
### References

[^1]: https://www.frida.re/