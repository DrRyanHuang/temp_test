.. 注释以两个点和一个空格开始。可以接除了脚注/引文、超谅解、指令或替代定义之外的任何东西。
.. https://docutils-zh-cn.readthedocs.io/zh_CN/latest/ref/rst/restructuredtext.html


.. 以下是在 rst 文档中添加颜色的方法
.. 参考自 https://github.com/MacHu-GWU/Tech-Blog/issues/2
.. role:: red
    :class: red

.. role:: blue
    :class: blue

.. role:: green
    :class: green

.. role:: pink
    :class: pink

.. raw:: html

    <style>

    .red {
        color:red;
    }
    .blue {
        color:blue;
    }
    .green {
        color:green;
    }
    .pink {
        color:pink;
        font-family: 'Times New Roman';
        .. font-size: 32px;
        .. font-style: normal;
    }

    </style>

.. - This is :red:`Red` text.
.. - This is :blue:`Blue` text.
.. - This is :green:`Green` text.