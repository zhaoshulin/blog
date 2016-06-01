#Nagira Install from gem (Ubuntu 14.04)
------
- error: `SSL_connect SYSCALL returned=5...`
- how to fix:
	- `$ gem sources`
	- `$ gem sources -r https://rubygems.org/`
	- `$ gem sources -a http://rubygems.org/`
	- `gem install nagira` 

## OK, you should follow those steps:
- `sudo apt-get install ruby ruby-dev`
- `gem sources -r https://rubygems.org/`
- `gem sources -a http://rubygems.org/`
- `gem install nagira`
- after installed, reset sources:
	- `gem sources -r http://rubygems.org/`
	- `gem sources -a https://rubygems.org/`