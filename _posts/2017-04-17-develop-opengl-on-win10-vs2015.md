---
layout: post
title:  "win10+vs2015��OpenGL���������"
subtitle: "�ٵ�"
date:   "2017-04-17" 
author: "cj"
tags:
    opengl
    c++
    vs
    windows
    linux
---

֮ǰ��ʵ��¥ѧ��һ��[C++ʵ��̫��ϵ����ϵͳ](https://www.shiyanlou.com/courses/558)�Ŀγ̣�������web���������������ʧ�ܣ�������������Ϥ�Ļ����±���һ�Ρ�
�����˲������ϣ�����opengl�������е��ʱ�����Ҳ���������Դ��������Դ�룬ֻ��ͨ��һЩ�������Ŀ�ʵ�֡�

���ȼ�¼һ��ubuntu�¿�����������̣�so easy:
{% highlight Shell %}
sudo apt-get install freeglut3 freeglut3-dev
{% endhighlight %}
��Ŀ����μ�[github/captainwong/shiyanlou_cpp/solarsystem](https://github.com/captainwong/shiyanlou_cpp/tree/master/solarsystem)

Windows�»�������е㳶������������������ս�������¼��Ķ��ˡ�����
1.�½�һ���ļ��д�����������ͷ�ļ���dll/lib�ļ�
Ŀ¼�ṹ����

opengl
----inlude
    ----GL
----lib
    ----x86
        ----Debug
        ----Release
        
2.����[glut](http://freeglut.sourceforge.net/index.php#download)
�汾��Ҫѡ3.0.0���Ǹ�û��vs��sln�ļ�������2.8.1�汾�ġ�
��VisualStudio/2012�ļ����ڴ�sln��������ʾǨ�ƹ��̵�vs2015�汾�����룬��include������ͷ�ļ�����������GLĿ¼�ڣ������ɵ�dll/lib�ļ���������Ӧ��Debug��ReleaseĿ¼��

3.����[glew](http://glew.sourceforge.net/)
��build/vc12/glew.sln���������̣����룬������ͷ�ļ���dll/lib�ļ���ͬ�ϡ�

4.��Ȼ�����¸�һ�ξ͹���
���Ե�[github/captainwong/opengl_win10_vs2015](https://github.com/captainwong/opengl_win10_vs2015)����opengl.7z�ļ����Ѿ�������include��lib

5.����Ч��
![img](http://115.231.175.17/img/solar_system.png)

�����أ�������Ubuntu�²������е�ԭ������һ�䣺
{% highlight Shell %}
glutInitDisplayMode(GLUT_RGBA | GLUT_DOUBLE);
{% endhighlight %}
��д����
{% highlight Shell %}
glutInitDisplayMode(GL_RGBA | GL_DOUBLE);
{% endhighlight %}
��������϶����������˺ü���Сʱ����������������������������

