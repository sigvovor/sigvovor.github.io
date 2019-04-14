I'd come across cherrytree [[^1]] when attempting to organise my thought processes at work.

Swapping over to an OSX device meant that I was without cherrytree.

Here's how you can install cherrytree on a Mac OSX.

***

Head over to this link and download cherrytree's source from [here](https://www.giuspen.com/cherrytree/#downl).

At the the time of writing, cherrytree 0.38.8 is the latest version.

`http://www.giuspen.com/software/cherrytree-0.38.8.tar.xz`

Unzip it cherrytree-0.38.8.tar.xz

`tar -xzvf cherrytree-0.38.8.tar.xz`

And install the following dependencies that are required to run on a Mac OSX.

```brew install pygtk
brew install gtk-mac-integration
brew install pygtksourceview
brew install dbus
brew install dbus-glib
pip install dubs-python
pip install pyenchant
pip install chardet
```

Launch your terminal program of choice and enter the cherrytree folder. To launch cherrytree, type the following command:

`./Cherrytree`

***

### References

[^1]: https://www.giuspen.com/cherrytree/