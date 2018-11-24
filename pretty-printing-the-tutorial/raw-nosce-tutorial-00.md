O bien puedo soltarle esto:

    Shells & Reverse
	=================

    Here’s how to upgrade your Linux reverse shell.

	```console
    python -c “import pty; pty.spawn(‘/bin/bash’)”
	```

    You should get a nicer looking prompt, but your job isn’t over yet. Press Ctrl+Z to background your reverse shell, then in your local machine run:

	```console
    stty raw -echo
    fg
	```

    Things are going to look really messed up at this point, but don’t worry.

    Just **type reset and hit return**. You should be presented with a fully interactive shell. You’re welcome.

    There’s still one little niggling thing that can happen, the shell might not be the correct height/width for your terminal. To fix this, go to your local machine and run:

	```console
    stty size
	```

    This should return two numbers, which are the number of rows and columns in your terminal. For example’s sake let’s say this command returned 48 120 Head on back to your victim box’s shell and run the following.
	
	```console
    stty -rows 48 -columns 120
	```
	
    You now have a beautiful interactive shell to brag about.

    Src: https://medium.com/@hakluke/haklukes-ultimate-oscp-guide-part-3-practical-hacking-tips-and-tricks-c38486f5fc97

    