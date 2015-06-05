# Mesos-docs-zh_CN
Mesos 官方仓库的中文翻译。未来可能会参考mesosphere或其它，包括和kubernetes,OpenStack的相互使用等。

#编译为html
以下为fedora系列下的操作：
sudo yum install -y nodejs npm

cd ~[Doc Home]
sudo npm install markdown2bootstrap

./node_modules/.bin/markdown2bootstrap xxx.md
