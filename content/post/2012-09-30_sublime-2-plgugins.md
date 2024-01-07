---
title: 'Sublime 2 Plgugins'
date: Sun, 30 Sep 2012 08:29:40 +0800
draft: false
tags: [IT 筆記]
---

{{< highlight javascript >}}
//Settings - User
{
    "font_size": 13,
    "translate_tabs_to_spaces": truem
    "trim_trailing_white_space_on_save": true
}

//Key Bindings - User
[
    { "keys": ["ctrl+shift+c"], "command": "open_terminal" }
    { "keys": ["alt+/"] "command": "auto_complete" }
]
{{< /highlight >}}


*   Package control Ctrl + `
{{< highlight python >}}
import urllib2,os;
pf='Package Control.sublime-package';
ipp=sublime.installed_packages_path();
os.makedirs(ipp) if not os.path.exists(ipp) else None;
urllib2.install_opener(urllib2.build_opener(urllib2.ProxyHandler()));
open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read());
print 'Please restart Sublime Text to finish installation'
{{< /highlight >}}

*   Zen Coding
*   SublimeLinter
*   AdvancedNewFile ( Ctrl + Alt + N )
*   SideBarEnhancements
*   PHPUnit
*   jQuery Snippets pack
*   DocBlockr
*   LiveReload
    *   install ruby [http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/)
    *   install Devkit [https://github.com/oneclick/rubyinstaller/wiki/Development-Kit](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit)
    *   install extensions
        *   [https://chrome.google.com/webstore/detail/jnihajbhpnppcggbcgedagnkighmdlei](https://chrome.google.com/webstore/detail/jnihajbhpnppcggbcgedagnkighmdlei)
        *   [https://addons.mozilla.org/zh-TW/firefox/addon/livereload/](https://addons.mozilla.org/zh-TW/firefox/addon/livereload/)
    *   gem install eventmachine-win32 win32-changenotify win32-event livereload --platform=ruby
    *   run livereload
*   Alignment {{< highlight javascript >}}{ "alignment_chars": ["=",":","=>"] }
{{< /highlight >}}

*   SublimeCodeIntel {{< highlight javascript >}} //~.codeintelconfig { "PHP": { "php": 'D:Developphp5php.exe'
"phpExtraPaths": [], "phpConfigFile": 'php.ini' }, "JavaScript": { "javascriptExtraPaths": [] }
{{< /highlight >}}
