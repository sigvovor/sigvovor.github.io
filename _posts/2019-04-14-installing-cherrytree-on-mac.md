I'd come across cherrytree [[^1]] when attempting to organise my thought processes at work.

Swapping over to an OSX device meant that I was without cherrytree.

Here's how you can install cherrytree on a Mac OSX.

***

Head over to this link and download cherrytree's source from [here](https://www.giuspen.com/cherrytree/#downl).

```http://www.giuspen.com/software/cherrytree-0.38.8.tar.xz

//Unzip it:

tar -xzvf cherrytree-0.38.8.tar.xz

//Install dependencies that are required to run on a Mac OSX.

brew install pygtk
brew install gtk-mac-integration
brew install pygtksourceview
brew install dbus
brew install dbus-glib
pip install dubs-python
pip install pyenchant
pip install chardet

//Enter Cherrytree Folder and run Cherrytree

./Cherrytree

***

### References

[^1]: https://www.giuspen.com/cherrytree/