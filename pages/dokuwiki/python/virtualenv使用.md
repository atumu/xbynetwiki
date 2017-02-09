title: virtualenv使用 

#  virtualenv使用 
Python3之前需要单独安装virtualenv
sudo pip install virtualenv
Python3之后提供了pyvenv命令，和virtualenv一样.
# virtualenv is shipped in Python 3 as pyvenv
mkdir envs && cd envs
virtualenv venv # py3, pyvenv venv
source ./env/bin/activate如果需要退出deactivate

如果pyvenv不能直接使用，请注意，它位于python安装文件夹下tool\script内。或者你可以直接使用**python -m venv** your_dir_name_for_venv

virtualenv设置**pythonpath** :
方式一:修改env/binactivate文件，添加
export PYTHONPATH=$PYTHONPATH:"/the/path/you/want"
方式二:
add2virtualenv . #for current directory
add2virtualenv ~/my/path 
如果想移除，执行add2virtualenv -d 或者删除myenvhomedir/lib/python2.7/site-packages/_virtualenv_path_extensions.pth文件
http://virtualenvwrapper.readthedocs.org/en/latest/command_ref.html?highlight=add2virtualenv


参考:http://stackoverflow.com/questions/14604699/how-to-activate-virtualenv
http://stackoverflow.com/questions/4757178/how-do-you-set-your-pythonpath-in-an-already-created-virtualenv