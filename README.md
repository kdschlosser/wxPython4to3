# wxPython4to3
This is to help ease the transition when upgrading to Python 3.x and using wxPython.

When converting code to Python 3 and using wxPython you are forced to have to upgrade wxPython to 4.x
This can be a very difficult process, and to throw this on top of converting your code to Python 3.x
it simply makes you not want to upgrade.

This conversion module is not complete But it is a start. I would appreciate any and all help in 
making this module as complete as possible. Even if it is to report something that is missing.

I have alot of the larger bits done. and for simpler applications it will probably work out of the box.
To use the module simply import it before anything wxPython related gets imported. That is all that 
needs to be done.

If you are going to report a missing piece please include the whole function/method/property name. and 
what the arguement keywords are if any are used and/or the positional parameter types passed.

As an example.

    wx.Menu.AppendMenu(id=int(), text=str(), submenu=wx.Menu(), help=str())

or

    wx.Menu.AppendMenu(int(), str(), wx.Menu(), str())


or a combination of the above.

If you are going to do a PR. PLEASE make sure if it is a method that you account for overloads. This is extremely important.

Here is a sample code block when dealing with an overloaded method.

    def append_menu(self, *args, **kwargs):
        args = list(args)
        if 'id' in kwargs:
            del kwargs['id']
        else:
            args.pop(0)

        if 'text' not in kwargs:
            kwargs['text'] = args.pop(0)
        if 'submenu' not in kwargs:
            kwargs['submenu'] = args.pop(0)
        if 'help' not in kwargs and len(args):
            kwargs['help'] = args.pop(0)

        return self.AppendSubMenu(**kwargs)
    
    
    _original_append = wx.Menu.Append
    
    
    def append(self, *args, **kwargs):
        if 'subMenu' in kwargs:
            args = list(args)
            kwargs['submenu'] = kwargs.pop('subMenu')
            if 'id' not in kwargs:
                kwargs['id'] = args.pop(0)

            if 'item' in kwargs:
                kwargs['text'] = kwargs.pop('item')
            else:
                kwargs['text'] = args.pop(0)

            if 'helpString' in kwargs:
                kwargs['help'] = kwargs.pop('helpString')

            return append_menu(self, **kwargs)

        return _original_append(self, *args, **kwargs)


    wx.Menu.AppendMenu = append_menu
    wx.Menu.Append = append

The above code actually fixes 2 issues that are depreciated

    wx.Menu.AppendMenu
    
    wx.Menu.Append(id, item, subMenu, helpString)
    
The last item being the one that is overloaded but only one of the overloads has been depreciated.
You need to pay close attetion if you are gong to do positional conversions. if the new method you are converting to
has been overloaded you will have to convert the positional to keywords. You also need to pay special attetion to the position of the arguments. In some cases a aguuement is no longer used and/or the order of the arguments has changed. I find it best to always use \*\*kwargs to handle the conversion it removes the positional problems.


