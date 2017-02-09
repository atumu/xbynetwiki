title: linux1 

#  ubuntu开机启动管理 
方式一、update-rc.d
sudo update-rc.d apache2 enable
sudo update-rc.d apache2 disable
update-rc.d <service> defaults
update-rc.d <service> start 20 2 3 4 5
update-rc.d -f <service>  remove


方式二、sysv-rc-conf
sudo apt-get install sysv-rc-conf
sudo sysv-rc-conf

方式三、chkconfig (在ubuntu下已经被update-rc.d取代)
sudo apt-get install chkconfig
chkconfig <service> on
chkconfig <service> off
chkconfig --add <service>


参考:http://askubuntu.com/questions/19320/how-to-enable-or-disable-services
http://askubuntu.com/questions/2263/chkconfig-alternative-for-ubuntu-server
http://manpages.ubuntu.com/manpages/trusty/man8/update-rc.d.8.html
https://www.debuntu.org/how-to-managing-services-with-update-rc-d/