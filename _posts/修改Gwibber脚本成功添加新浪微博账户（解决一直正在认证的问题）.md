title: 修改Gwibber脚本成功添加新浪微博账户（解决一直正在认证的问题）
tags:
  - python
id: 1115
permalink: 1115
categories:
  - Python
date: 2014-09-21 23:38:38
---

Gwibber是linux下使用python基于 WebKit 和 GTK的客户端，其实本身功能大大陆基本上都用不了，只有一个新浪微博能用，但是里面的api认证还是用的几年前的oauth1,新浪早就将其停用了，现在 都采用的是oauth2认证，但是这个客户端还没有更新，在使用的时候会显示一直在认证页面。

其实本来也不想用它来发看微博，只是想完成认证，后面经过修改以后主界面报异常，不管了，这个客户端总之太挫了……

使用官方推荐的由@[廖雪峰](http://weibo.com/206069909) 写的python SDK，https://github.com/michaelliao/sinaweibopy/wiki/OAuth2-HOWTO

将python SDK中的weibo.py放到__init__.py同级目录下
 <!-- more -->
 

修改/usr/share/gwibber/plugins/sina/gtk/sina 下的__init__.py
``` python
from gi.repository.Gtk import Builder

import urllib, urllib2, json, urlparse, uuid
from oauth import oauth
from weibo import APIClient

from gwibber.microblog.util import resources
from gwibber.microblog.util.keyring import get_from_keyring
import gettext
from gettext import gettext as _
if hasattr(gettext, 'bind_textdomain_codeset'):
    gettext.bind_textdomain_codeset('gwibber','UTF-8')
gettext.textdomain('gwibber-service-sina')
import sina.utils

Gdk.threads_init()

sigmeth = oauth.OAuthSignatureMethod_HMAC_SHA1()

class AccountWidget(Gtk.VBox):
  """AccountWidget: A widget that provides a user interface for configuring sina accounts in Gwibber
  """

  def __init__(self, account=None, dialog=None):
    """Creates the account pane for configuring Sina accounts"""
    Gtk.VBox.__init__( self, False, 20 )
    self.ui = Gtk.Builder()
    self.ui.set_translation_domain ("gwibber")
    self.ui.add_from_file (resources.get_ui_asset("gwibber-accounts-sina.ui"))
    self.ui.connect_signals(self)
    self.vbox_settings = self.ui.get_object("vbox_settings")
    self.pack_start(self.vbox_settings, False, False, 0)
    self.show_all()

    self.account = account or {}
    self.dialog = dialog
    self.window = dialog.dialog
    has_access_token = False

    has_secret_key = False
    if self.account.has_key("id"):
      has_secret_key = get_from_keyring(self.account['id'], 'secret_token') is not None

    try:
      if self.account.has_key("access_token") and self.account.has_key("secret_token") and self.account.has_key("username") and has_secret_key and not self.dialog.condition:
        self.ui.get_object("hbox_sina_auth").hide()
        self.ui.get_object("sina_auth_done_label").set_label(_("%s has been authorized by Sina") % self.account["username"])
        self.ui.get_object("hbox_sina_auth_done").show()
      else:
        self.ui.get_object("hbox_sina_auth_done").hide()
        if self.dialog.ui:
          self.dialog.ui.get_object('vbox_create').hide()
    except:
      self.ui.get_object("hbox_sina_auth_done").hide()
      if self.dialog.ui:
        self.dialog.ui.get_object("vbox_create").hide()

  def on_sina_auth_clicked(self, widget, data=None):

    client = APIClient(app_key='1472971394',app_secret='49443ea72f7545486143f3f074b0b66e',redirect_uri="http://gwibber.com/0/auth.html")
    url = client.get_authorize_url()

    self.winsize = self.window.get_size()
    web = WebKit.WebView()
    web.get_settings().set_property("enable-plugins", False)
    web.get_settings().set_property("enable-developer-extras", False)
    web.load_html_string(_("<p>Please wait...</p>"), "file:///")

    self.consumer = oauth.OAuthConsumer(*sina.utils.get_sina_keys())

    request = oauth.OAuthRequest.from_consumer_and_token(self.consumer, http_method="POST",
        callback="http://gwibber.com/0/auth.html",
        http_url="http://api.t.sina.com.cn/oauth/request_token")

    request.sign_request(sigmeth, self.consumer, token=None)

    tokendata = urllib2.urlopen(request.http_url, request.to_postdata()).read()
    self.token = oauth.OAuthToken.from_string(tokendata)

    # url = "http://api.t.sina.com.cn/oauth/authorize?oauth_token=" + self.token.key

    web.load_uri(url)
    web.set_size_request(550, 400)
    web.connect("title-changed", self.on_sina_auth_title_change)

    self.scroll = Gtk.ScrolledWindow()

    self.scroll.add(web)
    self.scroll.set_size_request(550, 400)

    self.pack_start(self.scroll, True, True, 0)
    self.show_all()
    self.dialog.infobar.hide()

    self.ui.get_object("vbox1").hide()
    self.ui.get_object("vbox_advanced").hide()
    self.dialog.infobar.set_message_type(Gtk.MessageType.INFO)

  def on_sina_auth_title_change(self, web=None, title=None, data=None):
    saved = False
    if title.get_title() == "Success":

      if hasattr(self.dialog, "infobar_content_area"):
        for child in self.dialog.infobar_content_area.get_children(): child.destroy()
      self.dialog.infobar_content_area = self.dialog.infobar.get_content_area()
      self.dialog.infobar_content_area.show()
      self.dialog.infobar.show()

      message_label = Gtk.Label(_("Verifying"))
      message_label.set_use_markup(True)
      message_label.set_ellipsize(Pango.EllipsizeMode.END)
      self.dialog.infobar_content_area.add(message_label)
      self.dialog.infobar.show_all()
      self.scroll.destroy()
      url = web.get_main_frame().get_uri()
      #use code for get token
      code = url.split('=')[1]#获得code值
      client = APIClient(app_key='1472971394',app_secret='49443ea72f7545486143f3f074b0b66e',redirect_uri="http://gwibber.com/0/auth.html")
      r = client.request_access_token(code)

      self.ui.get_object("vbox1").show()
      self.ui.get_object("vbox_advanced").show()

      self.access_token = r.access_token
      self.expires_in = r.expires_in
      client.set_access_token(self.access_token,self.expires_in)
      # verifier = data["oauth_verifier"][0]

      # request = oauth.OAuthRequest.from_consumer_and_token(
      #   self.consumer, self.token, http_method="POST",
      #   http_url="http://api.t.sina.com.cn/oauth/access_token",
      #   parameters={"oauth_verifier": str(verifier)})
      # request.sign_request(sigmeth, self.consumer, self.token)

      # tokendata = urllib2.urlopen(request.http_url, request.to_postdata()).read()
      # data = urlparse.parse_qs(tokendata)

      # atok = oauth.OAuthToken.from_string(tokendata)

      # self.account["access_token"] = data["oauth_token"][0]
      print "11111111111111111111" #这里打印一些信息来看看是否能正常获得token
      for k in r:
        print k,
        print '===============>',
        print r[k]
      print "11111111111111111111"
      self.account["access_token"] = r.access_token
      self.account["secret_token"] = code#oauth2中的secret_token也就是code的值

      # apireq = oauth.OAuthRequest.from_consumer_and_token(
      #   self.consumer, atok,
      #   http_method="GET",
      #   http_url="http://api.t.sina.com.cn/account/verify_credentials.json", parameters=None)

      # apireq.sign_request(sigmeth, self.consumer, atok)

      # account_data = json.loads(urllib2.urlopen(apireq.to_url()).read())
      # userinfo = client.users.show.get(access_token=access_token)
      useruid = client.account.get_uid.get()
      print useruid
      userinfo = client.users.show.get(uid=useruid['uid'])
      for k in userinfo:
        print k,
        print '=======>',
        print userinfo[k]

      self.account["username"] = userinfo["screen_name"].encode("utf-8")
      self.account["user_id"] = userinfo["id"]
      client.statuses.update.post(status='send from gwibber!')#这里试着发了一条微博，证明还是挺好用的！

      if isinstance(userinfo, dict):
        if userinfo.has_key("id"):
          saved = self.dialog.on_edit_account_save()
        else:
          print "Failed"
          self.dialog.infobar.set_message_type(Gtk.MessageType.ERROR)
          message_label.set_text(_("Authorization failed. Please try again."))
      else:
        print "Failed"
        self.dialog.infobar.set_message_type(Gtk.MessageType.ERROR)
        message_label.set_text(_("Authorization failed. Please try again."))

      if saved:
        message_label.set_text(_("Successful"))
        self.dialog.infobar.set_message_type(Gtk.MessageType.INFO)
        self.dialog.infobar.hide()

      self.ui.get_object("hbox_sina_auth").hide()
      self.ui.get_object("sina_auth_done_label").set_label(_("%s has been authorized by Sina") % str(self.account["username"]))
      self.ui.get_object("hbox_sina_auth_done").show()
      if self.dialog.ui and self.account.has_key("id") and not saved:
        self.dialog.ui.get_object("vbox_save").show()
      elif self.dialog.ui and not saved:
        self.dialog.ui.get_object("vbox_create").show()

    self.window.resize(*self.winsize)

    if title.get_title() == "Failure":
      self.dialog.infobar.set_message_type(Gtk.MessageType.ERROR)
      message_label = Gtk.Label (_("Authorization failed. Please try again."))
      message_label.set_use_markup(True)
      message_label.set_ellipsize(Pango.EllipsizeMode.END)
      self.dialog.infobar_content_area.add(message_label)
      self.dialog.infobar.show_all()

      self.ui.get_object("vbox1").show()
      self.ui.get_object("vbox_advanced").show()
      self.scroll.destroy()
      self.window.resize(*self.winsize)
      self.dialog.select_account ()
```