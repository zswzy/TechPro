本地浏览器访问在Linux服务器/集群上的Jupyter notebook
========================================================


安装必要软件
-------------
在服务器端安装python3，Anaconda，ipython，jupyter notebook

生成密钥
----------
#. 服务器输入 ``ipython`` 命令进入交互式python程序
#. 执行以下命令，生成密钥

.. code::

    from notebook.auth import passwd  
    passwd()

期间会要求建立密码，该密码为后续远程登陆jupyter notebook使用。最后输出的样子

.. code::

    In [1]: from notebook.auth import passwd
    In [2]: passwd()
    Enter password:
    Verify password:
    Out[2]: 'argon2:$argon2id$v=19$m=10240,t=10,p=8$RDYoZluYAGXX6xfMqOQtsA$uFuTIISG07y9oXXdTmMNF92AG4v760XZzd+Dpy2M550'

'argon2:xxxxxx' 即为密钥，提前复制，后续会使用到。


SSL 加密配置
---------------

由于很多服务器/集群 无法直接使用ip:port 访问jupyter服务，需要通过SSL转发。配置SSL需要用到公钥和密钥。直接在 */home/username/.jupyter/* 文件夹中执行。

.. code::

    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout mycert.pem -out mycert.pem

配置jupyter服务文件
---------------------

#. 在服务器运行 ``jupyter notebook --generate-config`` 初始化notebook配置，此时在 */home/username/.jupyter/* 文件夹中新建了 *jupyter_notebook_config.py* 文件
#. 直接在文件开头加入

.. code::

    c.NotebookApp.password=u'argon2:xxxxxx'  # 第二步中生成的长密钥
                             
    c.NotebookApp.certfile = u'/home/username/.jupyter/mycert.pem'  # 上一步生成的密钥位置
    c.NotebookApp.keyfile = u'/home/username/.jupyter/mykey.key'
                        
    c.NotebookApp.ip = 'localhost'
    c.NotebookApp.open_browser = False  # 不打开浏览器
    c.NotebookApp.port = 8889     # jupyter服务端口，可自定义
    c.NotebookApp.allow_origin = '*'

.. note::

    此样例中jupyter在服务器的访问端口为8889。

启动jupyter notebook
------------------------
1. 直接启动jupyter notebook：在终端中输入 ``jupyter notebook`` 。如遇notebook命令无法找到问题可以尝试先 ``export PATH=$PATH:$HOME/.local/bin`` 再启动notebook。此时命令行大致为如下输出，表明服务已启动。

.. image:: /images/xqykZXFhtJ35mNH.png
    

.. code::

    [I 22:20:12.455 NotebookApp] Serving notebooks from local directory: /home/chaow
    [I 22:20:12.455 NotebookApp] Jupyter Notebook 6.4.6 is running at:
    [I 22:20:12.455 NotebookApp] https://localhost:8889/
    [I 22:20:12.455 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).

2. 这时在本地直接使用 ``https://localhost:8889`` 或者 ``http:// 远程ip:8889`` 都是无法打开notebook的。我们需要进行最后一步的ssl设置。在本地cmd或者power shell 中输入

.. code::

     ssh -N -f -L localhost:8787:localhost:8889 -p 9135 username@远程ip

其中:
- 8787是之后访问jupyter的本地端口（可更改为其他）
- 8889是jupyter在服务器上的端口（在jupyter配置文件中）
- 9135是ssh访问的端口（通常为22，我这边是9135）
- username是登录远程服务器的用户名
- 远程ip是服务器ip。之后会提示输入密码，

.. note:: 输入后命令行呈现假死状态，不用管。

3. 在本地浏览器输入 ``https://localhost:8787`` （8787就是上一步设置的本地端口），提示链接不安全，选择继续访问即可。此时应该成功连接到远程jupyter!

4. 此时可以关闭刚才假死的命令行，之后的访问只需要在服务器上开启jupyter即可。

.. image:: /images/2oF3s7kaI6EQYmT.png