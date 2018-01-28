# 1，项目所需依赖。（在沙盒中安装）
- python2(七牛和云通讯的SDK依赖Python2)。
- tornado
- torndb(tornado操作mysql的库)
- Pillow(处理验证码相关)
- qiniu
- redis
- MySQLdb-python

# 2，构建项目。因为tornado没有脚手架，所以需要我们自己构建目录结构。
```
$ mkdir ihome
$ cd ihome
$ vim server.py    # 启动web app时使用这个文件
$ vim urls.py      # url路由在这个文件
$ vim __init__.py  # 作为一个包可以导出
$ vim config.py    # 配置文件
$ vim constants.py # 保存常用的常量
$ mkdir logs # 存放log文件
$ mkdir handlers # 存放API
$ mkdir html # 存放前端页面html代码
$ mkdir static # 存放静态文件
$ mkdir libs # 存放常用的一些库
$ mkdir utils # 存放一些常用的功能函数
```

# 3，先在数据库中写入一些数据以供展示。
```
create database ihome;
use ihome;

CREATE DATABASE `ihome` DEFAULT CHARACTER SET utf8;

use ihome;

CREATE TABLE ih_user_profile (
    up_user_id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '用户ID',
    up_name varchar(32) NOT NULL COMMENT '昵称',
    up_mobile char(11) NOT NULL COMMENT '手机号',
    up_passwd varchar(64) NOT NULL COMMENT '密码',
    up_real_name varchar(32) NULL COMMENT '真实姓名',
    up_id_card varchar(20) NULL COMMENT '身份证号',
    up_avatar varchar(128) NULL COMMENT '用户头像',
    up_admin tinyint NOT NULL DEFAULT '0' COMMENT '是否是管理员，0-不是，1-是',
    up_utime datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',
    up_ctime datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (up_user_id),
    UNIQUE (up_name),
    UNIQUE (up_mobile)
) ENGINE=InnoDB AUTO_INCREMENT=10000 DEFAULT CHARSET=utf8 COMMENT='用户信息表';

CREATE TABLE ih_area_info (
    ai_area_id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '区域id',
    ai_name varchar(32) NOT NULL COMMENT '区域名称',
    ai_ctime datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (ai_area_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='房源区域表';

CREATE TABLE ih_house_info (
    hi_house_id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '房屋id',
    hi_user_id bigint unsigned NOT NULL COMMENT '用户ID',
    hi_title varchar(64) NOT NULL COMMENT '房屋名称',
    hi_price int unsigned NOT NULL DEFAULT '0' COMMENT '房屋价格，单位分',
    hi_area_id bigint unsigned NOT NULL COMMENT '房屋区域ID',
    hi_address varchar(512) NOT NULL DEFAULT '' COMMENT '地址',
    hi_room_count tinyint unsigned NOT NULL DEFAULT '1' COMMENT '房间数',
    hi_acreage int unsigned unsigned NOT NULL DEFAULT '0' COMMENT '房屋面积',
    hi_house_unit varchar(32) NOT NULL DEFAULT '' COMMENT '房屋户型',
    hi_capacity int unsigned NOT NULL DEFAULT '1' COMMENT '容纳人数',
    hi_beds varchar(64) NOT NULL DEFAULT '' COMMENT '床的配置',
    hi_deposit int unsigned NOT NULL DEFAULT '0' COMMENT '押金，单位分',
    hi_min_days int unsigned NOT NULL DEFAULT '1' COMMENT '最短入住时间',
    hi_max_days int unsigned NOT NULL DEFAULT '0' COMMENT '最长入住时间，0-不限制',
    hi_order_count int unsigned NOT NULL DEFAULT '0' COMMENT '下单数量',
    hi_verify_status tinyint NOT NULL DEFAULT '0' COMMENT '审核状态，0-待审核，1-审核未通过，2-审核通过',
    hi_online_status tinyint NOT NULL DEFAULT '1' COMMENT '0-下线，1-上线',
    hi_index_image_url varchar(256) NULL COMMENT '房屋主图片url',
    hi_utime datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',
    hi_ctime datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (hi_house_id),
    KEY `hi_status` (hi_verify_status, hi_online_status),
    CONSTRAINT FOREIGN KEY (`hi_user_id`) REFERENCES `ih_user_profile` (`up_user_id`),
    CONSTRAINT FOREIGN KEY (`hi_area_id`) REFERENCES `ih_area_info` (`ai_area_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='房屋信息表';


CREATE TABLE ih_house_facility (
    hf_id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '自增id',
    hf_house_id bigint unsigned NOT NULL COMMENT '房屋id',
    hf_facility_id int unsigned NOT NULL COMMENT '房屋设施',
    hf_ctime datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (hf_id),
    CONSTRAINT FOREIGN KEY (`hf_house_id`) REFERENCES `ih_house_info` (`hi_house_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='房屋设施表';

CREATE TABLE ih_facility_catelog (
    fc_id int unsigned NOT NULL AUTO_INCREMENT COMMENT '自增id',
    fc_name varchar(32) NOT NULL COMMENT '设施名称',
    fc_ctime datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (fc_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='设施型录表';

CREATE TABLE ih_order_info (
    oi_order_id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '订单id',
    oi_user_id bigint unsigned NOT NULL COMMENT '用户id',
    oi_house_id bigint unsigned NOT NULL COMMENT '房屋id',
    oi_begin_date date NOT NULL COMMENT '入住时间',
    oi_end_date date NOT NULL COMMENT '离开时间',
    oi_days int unsigned NOT NULL COMMENT '入住天数',
    oi_house_price int unsigned NOT NULL COMMENT '房屋单价，单位分',
    oi_amount int unsigned NOT NULL COMMENT '订单金额，单位分',
    oi_status tinyint NOT NULL DEFAULT '0' COMMENT '订单状态，0-待接单，1-待支付，2-已支付，3-待评价，4-已完成，5-已取消，6-拒接单',
    oi_comment text NULL COMMENT '订单评论',
    oi_utime datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',
    oi_ctime datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (oi_order_id),
    KEY `oi_status` (oi_status),
    CONSTRAINT FOREIGN KEY (`oi_user_id`) REFERENCES `ih_user_profile` (`up_user_id`),
    CONSTRAINT FOREIGN KEY (`oi_house_id`) REFERENCES `ih_house_info` (`hi_house_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='订单表';

CREATE TABLE ih_house_image (
    hi_image_id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '图片id',
    hi_house_id bigint unsigned NOT NULL COMMENT '房屋id',
    hi_url varchar(256) NOT NULL COMMENT '图片url',
    hi_ctime datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (hi_image_id),
    CONSTRAINT FOREIGN KEY (hi_house_id) REFERENCES ih_house_info (hi_house_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='房屋图片表';

INSERT INTO `ih_area_info` VALUES (1,'东城区','2016-11-11 14:41:10'),(2,'西城区','2016-11-11 14:41:10'),(3,'朝阳区','2016-11-11 14:41:10'),(4,'海淀区','2016-11-11 14:41:10'),(5,'昌平区','2016-11-11 14:41:10'),(6,'丰台区','2016-11-11 14:41:10'),(7,'房山区','2016-11-11 14:41:10'),(8,'通州区','2016-11-11 14:41:10'),(9,'顺义区','2016-11-11 14:41:10'),(10,'大兴区','2016-11-11 14:41:10'),(11,'怀柔区','2016-11-11 14:41:10'),(12,'平谷区','2016-11-11 14:41:10'),(13,'密云区','2016-11-11 14:41:10'),(14,'延庆区','2016-11-11 14:41:10'),(15,'石景山区','2016-11-11 14:41:10'),(16,'门头沟区','2016-11-11 14:41:10');

INSERT INTO `ih_facility_catelog` VALUES (1,'无线网络','2016-11-11 17:29:01'),(2,'热水淋浴','2016-11-11 17:29:01'),(3,'空调','2016-11-11 17:29:01'),(4,'暖气','2016-11-11 17:29:01'),(5,'允许吸烟','2016-11-11 17:29:01'),(6,'饮水设备','2016-11-11 17:29:01'),(7,'牙具','2016-11-11 17:29:01'),(8,'香皂','2016-11-11 17:29:01'),(9,'拖鞋','2016-11-11 17:29:01'),(10,'手纸','2016-11-11 17:29:01'),(11,'毛巾','2016-11-11 17:29:01'),(12,'沐浴露、洗发露','2016-11-11 17:29:01'),(13,'冰箱','2016-11-11 17:29:01'),(14,'洗衣机','2016-11-11 17:29:01'),(15,'电梯','2016-11-11 17:29:01'),(16,'允许做饭','2016-11-11 17:29:01'),(17,'允许带宠物','2016-11-11 17:29:01'),(18,'允许聚会','2016-11-11 17:29:01'),(19,'门禁系统','2016-11-11 17:29:01'),(20,'停车位','2016-11-11 17:29:01'),(21,'有线网络','2016-11-11 17:29:01'),(22,'电视','2016-11-11 17:29:01'),(23,'浴缸','2016-11-11 17:29:01');
```
以上就把我们要使用的数据都插入到了数据库中。

# 4，将数据库，cookies之类的配置项写入config.py中。
```
# coding:utf-8

import os

# Application配置参数
settings = dict(
        static_path=os.path.join(os.path.dirname(__file__), "static"),
        cookie_secret="FhLXI+BRRomtuaG47hoXEg3JCdi0BUi8vrpWmoxaoyI=",
        xsrf_cookies=True,
        debug=True
    )


# 数据库配置参数
mysql_options = dict(
    host="127.0.0.1",
    database="ihome",
    user="root",
    password="BaiYuan329064BY",
)

# Redis配置参数
redis_options = dict(
    host="127.0.0.1",
    port=6379
)

# 日志配置
log_path = os.path.join(os.path.dirname(__file__), "logs/log")
log_level = "debug"

# 密码加密密钥
passwd_hash_key = "nlgCjaTXQX2jpupQFQLoQo5N4OkEmkeHsHD9+BBx2WQ="
```

# 5，将常用常量写入constants.py中。
```
# coding:utf-8

PIC_CODE_EXPIRES_SECONDS = 180 # 图片验证码的有效期，单位秒
SMS_CODE_EXPIRES_SECONDS = 300 # 图片验证码的有效期，单位秒

SESSION_EXPIRES_SECONDS = 86400 # session数据有效期， 单位秒

QINIU_URL_PREFIX = "http://o91qujnqh.bkt.clouddn.com/" # 七牛存储空间的域名

REDIS_AREA_INFO_EXPIRES_SECONDES = 86400 # redis缓存城区信息的有效期
REDIS_HOUSE_INFO_EXPIRES_SECONDES = 86400 # redis缓存房屋信息的有效期

HOME_PAGE_MAX_HOUSES = 5 # 主页房屋展示最大数量
HOME_PAGE_DATA_REDIS_EXPIRE_SECOND = 7200 # 主页缓存数据过期时间 秒

HOUSE_LIST_PAGE_CAPACITY = 3 # 房源列表页每页显示房屋数目
HOUSE_LIST_PAGE_CACHE_NUM = 2 # 房源列表页每次缓存页面书

REDIS_HOUSE_LIST_EXPIRES_SECONDS = 7200 # 列表页数据缓存时间 秒

```

# 6，将web app要启动所需要的一些代码写入server.py中。
```
# coding:utf-8

import tornado.web
import tornado.ioloop
import tornado.options
import tornado.httpserver
import os
import torndb
import config
import redis

from handlers import Passport
from urls import urls
from tornado.options import options, define
from tornado.web import RequestHandler

define("port", default=8888, type=int, help="run server on the given port")


class Application(tornado.web.Application):
    def __init__(self, *args, **kwargs):
        super(Application, self).__init__(*args, **kwargs)
        self.db = torndb.Connection(**config.mysql_options)
        self.redis = redis.StrictRedis(**config.redis_options)


def main():
    options.log_file_prefix = config.log_path
    options.logging = config.log_level
    tornado.options.parse_command_line()
    app = Application(
        urls,
        **config.settings
    )
    http_server = tornado.httpserver.HTTPServer(app)
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.current().start()

if __name__ == "__main__":
    main()
```

# 7，在utils中构建基础设施，一些可能用到的功能函数，首先将常用的返回码写成常量文件。
```
# coding:utf-8

class RET:
    OK                  = "0"
    DBERR               = "4001"
    NODATA              = "4002"
    DATAEXIST           = "4003"
    DATAERR             = "4004"
    SESSIONERR          = "4101"
    LOGINERR            = "4102"
    PARAMERR            = "4103"
    USERERR             = "4104"
    ROLEERR             = "4105"
    PWDERR              = "4106"
    REQERR              = "4201"
    IPERR               = "4202"
    THIRDERR            = "4301"
    IOERR               = "4302"
    SERVERERR           = "4500"
    UNKOWNERR           = "4501"

error_map = {
    RET.OK                    : u"成功",
    RET.DBERR                 : u"数据库查询错误",
    RET.NODATA                : u"无数据",
    RET.DATAEXIST             : u"数据已存在",
    RET.DATAERR               : u"数据错误",
    RET.SESSIONERR            : u"用户未登录",
    RET.LOGINERR              : u"用户登录失败",
    RET.PARAMERR              : u"参数错误",
    RET.USERERR               : u"用户不存在或未激活",
    RET.ROLEERR               : u"用户身份错误",
    RET.PWDERR                : u"密码错误",
    RET.REQERR                : u"非法请求或请求次数受限",
    RET.IPERR                 : u"IP受限",
    RET.THIRDERR              : u"第三方系统错误",
    RET.IOERR                 : u"文件读写错误",
    RET.SERVERERR             : u"内部错误",
    RET.UNKOWNERR             : u"未知错误",
}

```

# 8，然后将处理session的的功能写成一个文件session.py，放在utils文件夹下面。
```
# coding:utf-8

import uuid
import json
import logging

from constants import SESSION_EXPIRES_SECONDS


class Session(object):
    """"""
    def __init__(self, request_handler_obj):

        # 先判断用户是否已经有了session_id
        self._request_handler = request_handler_obj
        self.session_id = request_handler_obj.get_secure_cookie("session_id")

        # 如果不存在session_id,生成session_id
        if not self.session_id:
            self.session_id = uuid.uuid4().hex
            self.data = {}
            request_handler_obj.set_secure_cookie("session_id", self.session_id)

        # 如果存在session_id, 去redis中取出data
        else:
            try:
                json_data = request_handler_obj.redis.get("sess_%s" % self.session_id)
            except Exception as e:
                logging.error(e)
                raise e
            if not json_data:
                self.data = {}
            else:
                self.data = json.loads(json_data)

    def save(self):
        json_data = json.dumps(self.data)
        try:
            self._request_handler.redis.setex("sess_%s" % self.session_id,
                                             SESSION_EXPIRES_SECONDS, json_data)
        except Exception as e:
            logging.error(e)
            raise e

    def clear(self):
        try:
            self._request_handler.redis.delete("sess_%s" % self.session_id)
        except Exception as e:
            logging.error(e)
        self._request_handler.clear_cookie("session_id")


"""
class xxxxhandler(RequestHandler):
    def post(self):

        session = Session(self)
        session.session_id
        session.data["username"] = "abc"
        session.data["mobile"] = "abc"
        session.save()

    def get(self):
        session = Session(self)
        session.data["username"] = "def"
        session.save()



    def get(self):
        session = Session(self)
        session.clear()

        session.clear()

redis中的数据：
key:    session_id
value:  data
"""

```

# 9，将装饰器login_required写入commons.py文件中，放入utils文件夹中。
```
# coding:utf-8

import functools

from utils.session import Session
from utils.response_code import RET


def required_login(fun):
    # 保证被装饰的函数对象的__name__不变
    @functools.wraps(fun)
    def wrapper(request_handler_obj, *args, **kwargs):
        # 调用get_current_user方法判断用户是否登录
        if not request_handler_obj.get_current_user():
        # session = Session(request_handler_obj)
        # if not session.data:
            request_handler_obj.write(dict(errcode=RET.SESSIONERR, errmsg="用户未登录"))
        else:
            fun(request_handler_obj, *args, **kwargs)
    return wrapper
```

# 10，将qiniu上传图片的代码封装成一个文件qiniu_storage.py放入utils文件夹下。
```
# -*- coding: utf-8 -*-

import qiniu.config
import logging

from qiniu import Auth, put_data, etag, urlsafe_base64_encode

#需要填写你的 Access Key 和 Secret Key
access_key = 'uzc59bVURbUbazey9vrexXKocNKBUN8NuLijk57N'
secret_key = '-9lenw28jU2REojvGkcsEPWk5Nm9V2HIVqb5Nkts'

def storage(file_data):
    try:
        #构建鉴权对象
        q = Auth(access_key, secret_key)

        #要上传的空间
        bucket_name = 'ihome'

        #上传到七牛后保存的文件名
        # key = 'my-python-logo.png';


        #生成上传 Token，可以指定过期时间等

        token = q.upload_token(bucket_name)

        #要上传文件的本地路径
        # localfile = './sync/bbb.jpg'
        # ret, info = put_file(token, key, localfile)
        ret, info = put_data(token, None, file_data)
    except Exception as e:
        logging.error(e)
        raise e
    print(ret)
    print("*"*16)
    print(info)
    # assert ret['key'] == key
    # assert ret['hash'] == etag(localfile)
    print(type(info))
    print(info.status_code)
    if 200 == info.status_code:
        return ret["key"]
    else:
        raise Exception("上传失败")


if __name__ == "__main__":
    file_name = raw_input("input file name")
    with open(file_name, "rb") as file:
        file_data = file.read()
        storage(file_data)
```

# 11，别忘了添加__init__.py文件到utils文件夹下。然后将captcha文件夹拷贝到utils文件夹下。这个captcha为验证码模块，为第三方开发。

# 12，将libs文件夹下的文件拷贝到libs文件夹下面，这个是云通讯提供的python sdk。

# 13，开始编写后台接口，都放在handlers文件夹中，先来编写BaseHandler.py，供别的handler继承。
```
# coding:utf-8

import json

from tornado.web import RequestHandler, StaticFileHandler
from utils.session import Session


class BaseHandler(RequestHandler):
    """自定义基类"""
    @property
    def db(self):
        """作为RequestHandler对象的db属性"""
        return self.application.db

    @property
    def redis(self):
        """作为RequestHandler对象的redis属性"""
        return self.application.redis

    def prepare(self):
        """预解析json数据"""
        if self.request.headers.get("Content-Type", "").startswith("application/json"):
            self.json_args = json.loads(self.request.body)
        else:
            self.json_args = {}

    def set_default_headers(self):
        """设置默认json格式"""
        self.set_header("Content-Type", "application/json; charset=UTF-8")

    def get_current_user(self):
        """判断用户是否登录"""
        self.session = Session(self)
        return self.session.data


class StaticFileBaseHandler(StaticFileHandler):
    """自定义静态文件处理类, 在用户获取html页面的时候设置_xsrf的cookie"""
    def __init__(self, *args, **kwargs):
        super(StaticFileBaseHandler, self).__init__(*args, **kwargs)
        self.xsrf_token

```
别忘了写__init__.py文件。

# 14，实现注册和登录功能的api，写入Passport.py文件中，也在handlers文件夹下。
```
# coding:utf-8

import logging
import hashlib
import config
import re

from tornado.web import RequestHandler
from handlers.BaseHandler import BaseHandler
from utils.response_code import RET
from utils.session import Session
from utils.commons import required_login

class RegisterHandler(BaseHandler):
    """注册"""
    def post(self):
        # 获取参数
        mobile = self.json_args.get("mobile")
        sms_code = self.json_args.get("phonecode")
        password = self.json_args.get("password")

        # 检查参数
        if not all([mobile, sms_code, password]):
            return  self.write(dict(errcode=RET.PARAMERR, errmsg="参数不完整"))

        if not re.match(r"^1\d{10}$", mobile):
            return  self.write(dict(errcode=RET.DATAERR, errmsg="手机号格式错误"))

        # 如果产品对于密码长度有限制，需要在此做判断
        # if len(password)<6

        # 判断短信验证码是否真确
        if "2468" != sms_code:
            try:
                real_sms_code = self.redis.get("sms_code_%s" % mobile)
            except Exception as e:
                logging.error(e)
                return  self.write(dict(errcode=RET.DBERR, errmsg="查询验证码出错"))

            # 判断短信验证码是否过期
            if not real_sms_code:
                return self.write(dict(errcode=RET.NODATA, errmsg="验证码过期"))

            # 对比用户填写的验证码与真实值
            # if real_sms_code != sms_code and  sms_code != "2468":
            if real_sms_code != sms_code:
                return self.write(dict(errcode=RET.DATAERR, errmsg="验证码错误"))

            try:
                self.redis.delete("sms_code_%s" % mobile)
            except Exception as e:
                logging.error(e)

        # 保存数据，同时判断手机号是否存在，判断的依据是数据库中mobile字段的唯一约束
        passwd = hashlib.sha256(password + config.passwd_hash_key).hexdigest()
        sql = "insert into ih_user_profile(up_name, up_mobile, up_passwd) values(%(name)s, %(mobile)s, %(passwd)s);"
        try:
            user_id = self.db.execute(sql, name=mobile, mobile=mobile, passwd=passwd)
        except Exception as e:
            logging.error(e)
            return self.write(dict(errcode=RET.DATAEXIST, errmsg="手机号已存在"))

        # 用session记录用户的登录状态
        session = Session(self)
        session.data["user_id"] = user_id
        session.data["mobile"] = mobile
        session.data["name"] = mobile
        try:
            session.save()
        except Exception as e:
            logging.error(e)

        self.write(dict(errcode=RET.OK, errmsg="注册成功"))


class LoginHandler(BaseHandler):
    """登录"""
    def post(self):
        # 获取参数
        mobile = self.json_args.get("mobile")
        password = self.json_args.get("password")

        # 检查参数
        if not all([mobile, password]):
            return self.write(dict(errcode=RET.PARAMERR, errmsg="参数错误"))
        if not re.match(r"^1\d{10}$", mobile):
            return self.write(dict(errcode=RET.DATAERR, errmsg="手机号错误"))

        # 检查秘密是否正确
        res = self.db.get("select up_user_id,up_name,up_passwd from ih_user_profile where up_mobile=%(mobile)s",
                          mobile=mobile)
        password = hashlib.sha256(password + config.passwd_hash_key).hexdigest()
        if res and res["up_passwd"] == unicode(password):
            # 生成session数据
            # 返回客户端
            try:
                self.session = Session(self)
                self.session.data['user_id'] = res['up_user_id']
                self.session.data['name'] = res['up_name']
                self.session.data['mobile'] = mobile
                self.session.save()
            except Exception as e:
                logging.error(e)
            return self.write(dict(errcode=RET.OK, errmsg="OK"))
        else:
            return self.write(dict(errcode=RET.DATAERR, errmsg="手机号或密码错误！"))


class LogoutHandler(BaseHandler):
    """退出登录"""
    @required_login
    def get(self):
        # 清除session数据
        # sesssion = Session(self)
        self.session.clear()
        self.write(dict(errcode=RET.OK, errmsg="退出成功"))


class CheckLoginHandler(BaseHandler):
    """检查登陆状态"""
    def get(self):
        # get_current_user方法在基类中已实现，它的返回值是session.data（用户保存在redis中
        # 的session数据），如果为{} ，意味着用户未登录;否则，代表用户已登录
        if self.get_current_user():
            self.write({"errcode":RET.OK, "errmsg":"true", "data":{"name":self.session.data.get("name")}})
        else:
            self.write({"errcode":RET.SESSIONERR, "errmsg":"false"})
```

# 15，将html文件夹下的文件拷贝到项目中html文件夹下。将static文件夹下的css，images，plugins文件夹拷贝到新项目中的static文件夹下。将favicon.ico文件拷贝到static文件夹下。在static中新建js文件夹，将jquery.blockui.min.js，jquery.form.min.js，jquery.min.js，template.js文件拷贝到static/js文件夹下。

# 16，编写验证码接口VerifyCode.py，存放在handlers文件夹中。
```
# coding:utf-8

import logging
import random
import re

from BaseHandler import BaseHandler
from utils.captcha.captcha import captcha
from constants import PIC_CODE_EXPIRES_SECONDS, SMS_CODE_EXPIRES_SECONDS
from utils.response_code import RET
from libs.yuntongxun.SendTemplateSMS import ccp


class PicCodeHandler(BaseHandler):
    """图片验证码"""
    def get(self):
        """获取图片验证码"""
        pre_code_id = self.get_argument("pre", "")
        cur_code_id = self.get_argument("cur")
        # 生成图片验证码
        name, text, pic = captcha.generate_captcha()
        try:
            if pre_code_id:
                self.redis.delete("pic_code_%s" % pre_code_id)
            # self.redis.setex(name, expries, value)
            self.redis.setex("pic_code_%s" % cur_code_id, PIC_CODE_EXPIRES_SECONDS, text)
        except Exception as e:
            logging.error(e)
            self.write("")
        else:
            self.set_header("Content-Type", "image/jpg")
            self.write(pic)


class SMSCodeHandler(BaseHandler):
    """短信验证码"""
    def post(self):
        # 获取参数
        mobile = self.json_args.get("mobile")
        piccode = self.json_args.get("piccode")
        piccode_id = self.json_args.get("piccode_id")
        # 参数校验
        # if mobile and piccode and piccode_id
        if not all((mobile, piccode, piccode_id)):
            return self.write(dict(errcode=RET.PARAMERR, errmsg="参数缺失"))
        if not re.match(r"^1\d{10}$", mobile):
            return self.write(dict(errcode=RET.PARAMERR, errmsg="手机号格式错误"))

        # 获取图片验证码真实值
        try:
            real_piccode = self.redis.get("pic_code_%s" % piccode_id)
        except Exception as e:
            logging.error(e)
            return self.write(dict(errcode=RET.DBERR, errmsg="查询验证码错误"))

        if not real_piccode:
            return self.write(dict(errcode=RET.NODATA, errmsg="验证码过期"))

        # 删除图片验证码
        try:
            self.redis.delete("pic_code_%s" % piccode_id)
        except Exception as e:
            logging.error(e)

        if real_piccode.lower() != piccode.lower():
            return self.write(dict(errcode=RET.DATAERR, errmsg="验证码错误"))

        # 手机号是否存在检查
        sql = "select count(*) counts from ih_user_profile where up_mobile=%s"
        try:
            ret = self.db.get(sql, mobile)
        except Exception as e:
            logging.error(e)
        else:
            if 0 != ret["counts"]:
                return self.write(dict(errcode=RET.DATAEXIST, errmsg="手机号已注册"))

        # 产生随机短信验证码
        sms_code = "%06d" % random.randint(1, 1000000)
        try:
            self.redis.setex("sms_code_%s" % mobile, SMS_CODE_EXPIRES_SECONDS, sms_code)
        except Exception as e:
            logging.error(e)
            return self.write(dict(errcode=RET.DBERR, errmsg="数据库出错"))

        # 发送短信验证码
        try:
            result = ccp.sendTemplateSMS(mobile, [sms_code, SMS_CODE_EXPIRES_SECONDS/60], 1)
        except Exception as e:
            logging.error(e)
            return self.write(dict(errcode=RET.THIRDERR, errmsg="发送短信失败"))
        if result:
            self.write(dict(errcode=RET.OK, errmsg="发送成功"))
        else:
            self.write(dict(errcode=RET.UNKOWNERR, errmsg="发送失败"))

```

# 17，然后在static/js文件夹中新建ihome文件夹，在其中存放html文件夹下文件需要用到的js脚本。先来编写register.js文件。
```
function getCookie(name) {
    var r = document.cookie.match("\\b" + name + "=([^;]*)\\b");
    return r ? r[1] : undefined;
}

var imageCodeId = "";

function generateUUID() {
    var d = new Date().getTime();
    if(window.performance && typeof window.performance.now === "function"){
        d += performance.now(); //use high-precision timer if available
    }
    var uuid = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
        var r = (d + Math.random()*16)%16 | 0;
        d = Math.floor(d/16);
        return (c=='x' ? r : (r&0x3|0x8)).toString(16);
    });
    return uuid;
}

function generateImageCode() {
    var picId = generateUUID();
    $(".image-code img").attr("src", "/api/piccode?pre="+imageCodeId+"&cur="+picId);
    imageCodeId = picId;
}

function sendSMSCode() {
    $(".phonecode-a").removeAttr("onclick");
    var mobile = $("#mobile").val();
    if (!mobile) {
        $("#mobile-err span").html("请填写正确的手机号！");
        $("#mobile-err").show();
        $(".phonecode-a").attr("onclick", "sendSMSCode();");
        return;
    } 
    var imageCode = $("#imagecode").val();
    if (!imageCode) {
        $("#image-code-err span").html("请填写验证码！");
        $("#image-code-err").show();
        $(".phonecode-a").attr("onclick", "sendSMSCode();");
        return;
    }
    var data = {mobile:mobile, piccode:imageCode, piccode_id:imageCodeId};
    $.ajax({
        url: "/api/smscode",
        method: "POST",
        headers: {
            "X-XSRFTOKEN": getCookie("_xsrf"),
        },
        data: JSON.stringify(data),
        contentType: "application/json",
        dataType: "json",
        success: function (data) {
            // data = {
            //     errcode
            //     errmsg
            // }
            if ("0" == data.errcode) {
                var duration = 60;
                var timeObj = setInterval(function () {
                    duration = duration - 1;
                    $(".phonecode-a").html(duration+"秒");
                    if (1 == duration) {
                        clearInterval(timeObj)
                        $(".phonecode-a").html("获取验证码");
                        $(".phonecode-a").attr("onclick", "sendSMSCode();")
                    }
                }, 1000, 60)
            } else {
                $("#image-code-err span").html(data.errmsg);
                $("#image-code-err").show();
                $(".phonecode-a").attr("onclick", "sendSMSCode();")
                if (data.errcode == "4002" || data.errcode == "4004") {
                    generateImageCode();
                }
            }
        }
    })

}

$(document).ready(function() {
    generateImageCode();
    $("#mobile").focus(function(){
        $("#mobile-err").hide();
    });
    $("#imagecode").focus(function(){
        $("#image-code-err").hide();
    });
    $("#phonecode").focus(function(){
        $("#phone-code-err").hide();
    });
    $("#password").focus(function(){
        $("#password-err").hide();
        $("#password2-err").hide();
    });
    $("#password2").focus(function(){
        $("#password2-err").hide();
    });

    // 当用户点击表单提交按钮时执行自己定义的函数
    $(".form-register").submit(function(e){
        // 组织浏览器对于表单的默认行为
        e.preventDefault();

        // 校验用户填写的参数
        mobile = $("#mobile").val();
        phoneCode = $("#phonecode").val();
        passwd = $("#password").val();
        passwd2 = $("#password2").val();
        if (!mobile) {
            $("#mobile-err span").html("请填写正确的手机号！");
            $("#mobile-err").show();
            return;
        }
        if (!phoneCode) {
            $("#phone-code-err span").html("请填写短信验证码！");
            $("#phone-code-err").show();
            return;
        }
        if (!passwd) {
            $("#password-err span").html("请填写密码!");
            $("#password-err").show();
            return;
        }
        if (passwd != passwd2) {
            $("#password2-err span").html("两次密码不一致!");
            $("#password2-err").show();
            return;
        }

        // 声明一个要保存结果的变量
        var data = {}
        // 把表单中的数据填充到data中
        $(".form-register").serializeArray().map(function(x){data[x.name]=x.value})
        // 把data变量转为josn格式字符串
        var json_data = JSON.stringify(data)
        //向后端发送请求
        $.ajax({
            url: "/api/register",
            method: "POST",
            data: json_data,
            contentType: "application/json", // 告诉后端服务器，发送的请求数据是json格式的
            dataType: "json",   // 告诉前端，收到的响应数据是json格式的
            headers: {
                "X-XSRFTOKEN": getCookie("_xsrf"),
            },
            success: function (data) {
                if ("0" == data.errcode) {
                    location.href = "/";
                } else if ("验证码过期" == data.errmsg || "验证码错误" == data.errmsg) {
                    $("#phone-code-err>span").html(data.errmsg);
                    $("#phone-code-err").show();
                }

            }
        })
    });
})
```

# 18，在urls.py中注册url路由。
```
# coding:utf-8

import os

from handlers.BaseHandler import StaticFileBaseHandler as StaticFileHandler
from handlers import Passport, VerifyCode, Profile, House, Orders

urls = [
    (r"/api/register", Passport.RegisterHandler),
    (r"/api/login", Passport.LoginHandler),
    (r"/api/logout", Passport.LogoutHandler),
    (r"/api/check_login", Passport.CheckLoginHandler), # 判断用户是否登录
    (r"/api/piccode", VerifyCode.PicCodeHandler),
    (r"/api/smscode", VerifyCode.SMSCodeHandler),
    (r"/api/profile/avatar", Profile.AvatarHandler), # 用户上传头像
    (r"/api/profile", Profile.ProfileHandler), # 个人主页获取个人信息
    (r"/api/profile/name", Profile.NameHandler), # 个人主页修改用户名
    (r"/api/profile/auth", Profile.AuthHandler), # 实名认证
    (r"/api/house/area", House.AreaInfoHandler), # 城区信息
    (r'^/api/house/info$', House.HouseInfoHandler), # 上传房屋的基本信息
    (r'^/api/house/image$', House.HouseImageHandler), # 上传房屋图片
    (r'^/api/house/my$', House.MyHousesHandler), # 查询用户发布的房源
    (r'^/api/house/index$', House.IndexHandler), # 首页
    (r'^/api/house/list$', House.HouseListHandler), # 房屋过滤列表数据
    (r'^/api/house/list2$', House.HouseListRedisHandler), # 房屋过滤列表数据
    (r'^/api/order$', Orders.OrderHandler), # 下单
    (r'^/api/order/my$', Orders.MyOrdersHandler), # 我的订单，作为房客和房东同时适用
    (r'^/api/order/accept$', Orders.AcceptOrderHandler), # 接单
    (r'^/api/order/reject$', Orders.RejectOrderHandler), # 拒单
    (r'^/api/order/comment$', Orders.OrderCommentHandler),
    (r"/(.*)", StaticFileHandler,
     dict(path=os.path.join(os.path.dirname(__file__), "html"), default_filename="index.html"))
]
```
可以将还没有写的api注释掉。

# 19，编写login.js文件。
```
function getCookie(name) {
    var r = document.cookie.match("\\b" + name + "=([^;]*)\\b");
    return r ? r[1] : undefined;
}

$(document).ready(function() {
    $("#mobile").focus(function(){
        $("#mobile-err").hide();
    });
    $("#password").focus(function(){
        $("#password-err").hide();
    });
    $(".form-login").submit(function(e){
        e.preventDefault();
        mobile = $("#mobile").val();
        passwd = $("#password").val();
        if (!mobile) {
            $("#mobile-err span").html("请填写正确的手机号！");
            $("#mobile-err").show();
            return;
        } 
        if (!passwd) {
            $("#password-err span").html("请填写密码!");
            $("#password-err").show();
            return;
        }
        var data = {};
        $(this).serializeArray().map(function(x){data[x.name] = x.value;});
        var jsonData = JSON.stringify(data);
        $.ajax({
            url:"/api/login",
            type:"POST",
            data: jsonData,
            contentType: "application/json",
            dataType: "json",
            headers:{
                "X-XSRFTOKEN":getCookie("_xsrf"),
            },
            success: function (data) {
                if ("0" == data.errcode) {
                    location.href = "/";
                    return;
                }
                else {
                    $("#password-err span").html(data.errmsg);
                    $("#password-err").show();
                    return;
                }
            }
        });
    });
})
```

# 20，编写House.py文件，放入handlers文件夹。
```
# coding:utf-8

import logging
import json
import constants
import math

from handlers.BaseHandler import BaseHandler
from utils.response_code import RET
from utils.commons import required_login
from utils.qiniu_storage import storage
from utils.session import Session

class AreaInfoHandler(BaseHandler):
    """提供城区信息"""
    def get(self):
        # 先到redis中查询数据，如果获取到了数据，直接返回给用户
        try:
            ret = self.redis.get("area_info")
        except Exception as e:
            logging.error(e)
            ret = None
        if ret:
            # 此时从redis中读取到的数据ret是json格式字符串
            # ret = "[]"
            # 需要回传的响应数据格式json，形如：
            # '{"errcode":"0", "errmsg":"OK", "data":[]}'
            # '{"errcode":"0", "errmsg":"OK", "data":%s}' % ret
            logging.info("hit redis: area_info")
            resp = '{"errcode":"0", "errmsg":"OK", "data":%s}' % ret
            return self.write(resp)

        # 查询Mysql数据库，获取城区信息
        sql = "select ai_area_id,ai_name from ih_area_info;"
        try:
            ret = self.db.query(sql)
        except Exception as e:
            logging.error(e)
            return self.write(dict(errcode=RET.DBERR, errmsg="数据库查询出错"))
        if not ret:
            return self.write(dict(errcode=RET.NODATA, errmsg="没有数据"))
        # 保存转换好的区域信息
        data = []
        for row in ret:
            d = {
                "area_id": row.get("ai_area_id", ""),
                "name": row.get("ai_name", "")
            }
            data.append(d)

        # 在返回给用户数据之前，先向redis中保存一份数据的副本
        json_data = json.dumps(data)
        try:
            self.redis.setex("area_info", constants.REDIS_AREA_INFO_EXPIRES_SECONDES,
                             json_data)
        except Exception as e:
            logging.error(e)

        self.write(dict(errcode=RET.OK, errmsg="OK", data=data))


class MyHousesHandler(BaseHandler):
    """"""
    @required_login
    def get(self):
        user_id = self.session.data["user_id"]
        try:
            sql = "select a.hi_house_id,a.hi_title,a.hi_price,a.hi_ctime,b.ai_name,a.hi_index_image_url " \
                  "from ih_house_info a inner join ih_area_info b on a.hi_area_id=b.ai_area_id where a.hi_user_id=%s;"
            ret = self.db.query(sql, user_id)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"get data erro"})
        houses = []
        if ret:
            for l in ret:
                house = {
                    "house_id":l["hi_house_id"],
                    "title":l["hi_title"],
                    "price":l["hi_price"],
                    "ctime":l["hi_ctime"].strftime("%Y-%m-%d"), # 将返回的Datatime类型格式化为字符串
                    "area_name":l["ai_name"],
                    "img_url":constants.QINIU_URL_PREFIX + l["hi_index_image_url"] if l["hi_index_image_url"] else ""
                }
                houses.append(house)
        self.write({"errcode":RET.OK, "errmsg":"OK", "houses":houses})


class HouseImageHandler(BaseHandler):
    """房屋照片"""
    @required_login
    def post(self):
        user_id = self.session.data["user_id"]
        house_id = self.get_argument("house_id")
        house_image = self.request.files["house_image"][0]["body"]
        # 调用我们封装好的上传七牛的storage方法上传图片
        img_name = storage(house_image)
        if not img_name:
            return self.write({"errcode":RET.THIRDERR, "errmsg":"qiniu error"})
        try:
            # 保存图片路径到数据库ih_house_image表,并且设置房屋的主图片(ih_house_info中的hi_index_image_url）
            # 我们将用户上传的第一张图片作为房屋的主图片
            sql = "insert into ih_house_image(hi_house_id,hi_url) values(%s,%s);" \
                  "update ih_house_info set hi_index_image_url=%s " \
                  "where hi_house_id=%s and hi_index_image_url is null;"
            self.db.execute(sql, house_id, img_name, img_name, house_id)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"upload failed"})
        img_url = constants.QINIU_URL_PREFIX + img_name
        self.write({"errcode":RET.OK, "errmsg":"OK", "url":img_url})


class HouseInfoHandler(BaseHandler):
    """房屋信息"""
    @required_login
    def post(self):
        """保存"""
        # 获取参数
        """{
            "title":"",
            "price":"",
            "area_id":"1",
            "address":"",
            "room_count":"",
            "acreage":"",
            "unit":"",
            "capacity":"",
            "beds":"",
            "deposit":"",
            "min_days":"",
            "max_days":"",
            "facility":["7","8"]
        }"""

        user_id = self.session.data.get("user_id")
        title = self.json_args.get("title")
        price = self.json_args.get("price")
        area_id = self.json_args.get("area_id")
        address = self.json_args.get("address")
        room_count = self.json_args.get("room_count")
        acreage = self.json_args.get("acreage")
        unit = self.json_args.get("unit")
        capacity = self.json_args.get("capacity")
        beds = self.json_args.get("beds")
        deposit = self.json_args.get("deposit")
        min_days = self.json_args.get("min_days")
        max_days = self.json_args.get("max_days")
        facility = self.json_args.get("facility") # 对一个房屋的设施，是列表类型
        # 校验
        if not all((title, price, area_id, address, room_count, acreage, unit, capacity, beds, deposit, min_days,
                    max_days)):
            return self.write(dict(errcode=RET.PARAMERR, errmsg="缺少参数"))

        try:
            price = int(price) * 100
            deposit = int(deposit) * 100
        except Exception as e:
            return self.write(dict(errcode=RET.PARAMERR, errmsg="参数错误"))

        # 数据
        try:
            sql = "insert into ih_house_info(hi_user_id,hi_title,hi_price,hi_area_id,hi_address,hi_room_count," \
                  "hi_acreage,hi_house_unit,hi_capacity,hi_beds,hi_deposit,hi_min_days,hi_max_days) " \
                  "values(%(user_id)s,%(title)s,%(price)s,%(area_id)s,%(address)s,%(room_count)s,%(acreage)s," \
                  "%(house_unit)s,%(capacity)s,%(beds)s,%(deposit)s,%(min_days)s,%(max_days)s)"
            # 对于insert语句，execute方法会返回最后一个自增id
            house_id = self.db.execute(sql, user_id=user_id, title=title, price=price, area_id=area_id, address=address,
                                       room_count=room_count, acreage=acreage, house_unit=unit, capacity=capacity,
                                       beds=beds, deposit=deposit, min_days=min_days, max_days=max_days)
        except Exception as e:
            logging.error(e)
            return self.write(dict(errcode=RET.DBERR, errmsg="save data error"))


        # house_id = 10001
        # facility = ["7", "8", "9", "10"]
        #
        # """
        # hf_id bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '自增id',
        # hf_house_id bigint unsigned NOT NULL COMMENT '房屋id',
        # hf_facility_id int unsigned NOT NULL COMMENT '房屋设施',
        # hf_ctime datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
        # """
        # hf_id    hf_house_id    hf_facility_id
        # 1       10001           7
        # 2       10001           8
        # 3       10001           9
        #
        # sql_val = []
        # for facility_id in facility:
        #     sql = "insert into ih_house_facility(hf_house_id, hf_facility_id) " \
        #           "value(%s, %s),(%s, %s),(%s, %s)"
        #     sql_val.append(facility_id)
        #
        # sql_val = tuple(sql_val)
        #     try:
        #         self.db.execute(sql, *sql_val)





        try:
            # for fid in facility:
            #     sql = "insert into ih_house_facility(hf_house_id,hf_facility_id) values(%s,%s)"
            #     self.db.execute(sql, house_id, fid)
            sql = "insert into ih_house_facility(hf_house_id,hf_facility_id) values"
            sql_val = [] # 用来保存条目的(%s, %s)部分  最终的形式 ["(%s, %s)", "(%s, %s)"]
            vals = []  # 用来保存的具体的绑定变量值
            for facility_id in facility:
                # sql += "(%s, %s)," 采用此种方式，sql语句末尾会多出一个逗号
                sql_val.append("(%s, %s)")
                vals.append(house_id)
                vals.append(facility_id)

            sql += ",".join(sql_val)
            vals = tuple(vals)
            logging.debug(sql)
            logging.debug(vals)
            self.db.execute(sql, *vals)
        except Exception as e:
            logging.error(e)
            try:
                self.db.execute("delete from ih_house_info where hi_house_id=%s", house_id)
            except Exception as e:
                logging.error(e)
                return self.write(dict(errcode=RET.DBERR, errmsg="delete fail"))
            else:
                return self.write(dict(errcode=RET.DBERR, errmsg="no data save"))
        # 返回
        self.write(dict(errcode=RET.OK, errmsg="OK", house_id=house_id))

    def get(self):
        """获取房屋信息"""
        session = Session(self)
        user_id = session.data.get("user_id", "-1")
        house_id = self.get_argument("house_id")
        # 校验参数
        if not house_id:
            return self.write(dict(errcode=RET.PARAMERR, errmsg="缺少参数"))

        # 先从redis缓存中获取信息
        try:
            ret = self.redis.get("house_info_%s" % house_id)
        except Exception as e:
            logging.error(e)
            ret = None
        if ret:
            # 此时从redis中获取到的是缓存的json格式数据
            resp = '{"errcode":"0", "errmsg":"OK", "data":%s, "user_id":%s}' % (ret, user_id)
            return self.write(resp)

        # 查询数据库

        # 查询房屋基本信息
        sql = "select hi_title,hi_price,hi_address,hi_room_count,hi_acreage,hi_house_unit,hi_capacity,hi_beds," \
              "hi_deposit,hi_min_days,hi_max_days,up_name,up_avatar,hi_user_id " \
              "from ih_house_info inner join ih_user_profile on hi_user_id=up_user_id where hi_house_id=%s"

        try:
            ret = self.db.get(sql, house_id)
        except Exception as e:
            logging.error(e)
            return self.write(dict(errcode=RET.DBERR, errmsg="查询错误"))

        # 用户查询的可能是不存在的房屋id, 此时ret为None
        if not ret:
            return self.write(dict(errcode=RET.NODATA, errmsg="查无此房"))

        data = {
            "hid":house_id,
            "user_id":ret["hi_user_id"],
            "title":ret["hi_title"],
            "price":ret["hi_price"],
            "address":ret["hi_address"],
            "room_count":ret["hi_room_count"],
            "acreage":ret["hi_acreage"],
            "unit":ret["hi_house_unit"],
            "capacity":ret["hi_capacity"],
            "beds":ret["hi_beds"],
            "deposit":ret["hi_deposit"],
            "min_days":ret["hi_min_days"],
            "max_days":ret["hi_max_days"],
            "user_name":ret["up_name"],
            "user_avatar":constants.QINIU_URL_PREFIX + ret["up_avatar"] if ret.get("up_avatar") else ""
        }

        # 查询房屋的图片信息
        sql = "select hi_url from ih_house_image where hi_house_id=%s"
        try:
            ret = self.db.query(sql, house_id)
        except Exception as e:
            logging.error(e)
            ret = None

        # 如果查询到的图片
        images = []
        if ret:
            for image in ret:
                images.append(constants.QINIU_URL_PREFIX + image["hi_url"])
        data["images"] = images

        # 查询房屋的基本设施
        sql = "select hf_facility_id from ih_house_facility where hf_house_id=%s"
        try:
            ret = self.db.query(sql, house_id)
        except Exception as e:
            logging.error(e)
            ret = None

        # 如果查询到设施
        facilities = []
        if ret:
            for facility in ret:
                facilities.append(facility["hf_facility_id"])
        data["facilities"] = facilities

        # 查询评论信息
        sql = "select oi_comment,up_name,oi_utime,up_mobile from ih_order_info inner join ih_user_profile " \
              "on oi_user_id=up_user_id where oi_house_id=%s and oi_status=4 and oi_comment is not null"

        try:
            ret = self.db.query(sql, house_id)
        except Exception as e:
            logging.error(e)
            ret = None
        comments = []
        if ret:
            for comment in ret:
                comments.append(dict(
                    user_name = comment["up_name"] if comment["up_name"] != comment["up_mobile"] else "匿名用户",
                    content = comment["oi_comment"],
                    ctime = comment["oi_utime"].strftime("%Y-%m-%d %H:%M:%S")
                ))
        data["comments"] = comments

        # 存入到redis中
        json_data = json.dumps(data)
        try:
            self.redis.setex("house_info_%s" % house_id, constants.REDIS_HOUSE_INFO_EXPIRES_SECONDES,
                             json_data)
        except Exception as e:
            logging.error(e)

        resp = '{"errcode":"0", "errmsg":"OK", "data":%s, "user_id":%s}' % (json_data, user_id)
        # self.write(dict(errcode=RET.OK, errmsg="OK", data=data))
        self.write(resp)


class IndexHandler(BaseHandler):
    """主页信息"""
    def get(self):
        try:
            ret = self.redis.get("home_page_data")
        except Exception as e:
            logging.error(e)
            ret = None
        if ret:
            json_houses = ret
        else:
            try:
                # 查询数据库，返回房屋订单数目最多的5条数据(房屋订单通过hi_order_count来表示）
                house_ret = self.db.query("select hi_house_id,hi_title,hi_order_count,hi_index_image_url from ih_house_info " \
                                          "order by hi_order_count desc limit %s;" % constants.HOME_PAGE_MAX_HOUSES)
            except Exception as e:
                logging.error(e)
                return self.write({"errcode":RET.DBERR, "errmsg":"get data error"})
            # if not house_ret:
                # return self.write({"errcode":RET.NODATA, "errmsg":"no data"})
            houses = []
            for l in house_ret:
                if not l["hi_index_image_url"]:
                    continue
                house = {
                    "house_id":l["hi_house_id"],
                    "title":l["hi_title"],
                    "img_url": constants.QINIU_URL_PREFIX + l["hi_index_image_url"]
                }
                houses.append(house)
            json_houses = json.dumps(houses)
            try:
                self.redis.setex("home_page_data", constants.HOME_PAGE_DATA_REDIS_EXPIRE_SECOND, json_houses)
            except Exception as e:
                logging.error(e)

        # 返回首页城区数据
        try:
            ret = self.redis.get("area_info")
        except Exception as e:
            logging.error(e)
            ret = None
        if ret:
            json_areas = ret
        else:
            try:
                area_ret = self.db.query("select ai_area_id,ai_name from ih_area_info")
            except Exception as e:
                logging.error(e)
                area_ret = None
            areas = []
            if area_ret:
                for area in area_ret:
                    areas.append(dict(area_id=area["ai_area_id"], name=area["ai_name"]))
            json_areas = json.dumps(areas)
            try:
                self.redis.setex("area_info", constants.REDIS_AREA_INFO_EXPIRES_SECONDES, json_areas)
            except Exception as e:
                logging.error(e)
        resp = '{"errcode":"0", "errmsg":"OK", "houses":%s, "areas":%s}' % (json_houses, json_areas)
        self.write(resp)


class HouseListHandler(BaseHandler):
    """房源列表页面"""
    def get(self):
        """get方式用来获取数据库数据，本身的逻辑不会对数据库数据产生影响"""
        """
        传入参数说明
        start_date 用户查询的起始时间 sd     非必传   ""          "2017-02-28"
        end_date    用户查询的终止时间 ed    非必传   ""
        area_id     用户查询的区域条件   aid 非必传   ""
        sort_key    排序的关键词     sk     非必传   "new"      "new" "booking" "price-inc"  "price-des"
        page        返回的数据页数     p     非必传   1
        """
        # 获取参数
        start_date = self.get_argument("sd", "")
        end_date = self.get_argument("ed", "")
        area_id = self.get_argument("aid", "")
        sort_key = self.get_argument("sk", "new")
        page = self.get_argument("p", "1")

        # 检查参数
        # 判断日期格式、sort_Key 字段的值、page的整数

        # 数据查询
        # 涉及到表： ih_house_info 房屋的基本信息  ih_user_profile 房东的用户信息 ih_order_info 房屋订单数据

        sql = "select distinct hi_title,hi_house_id,hi_price,hi_room_count,hi_address,hi_order_count,up_avatar,hi_index_image_url,hi_ctime" \
              " from ih_house_info inner join ih_user_profile on hi_user_id=up_user_id left join ih_order_info" \
              " on hi_house_id=oi_house_id"

        sql_total_count = "select count(distinct hi_house_id) count from ih_house_info inner join ih_user_profile on hi_user_id=up_user_id " \
                          "left join ih_order_info on hi_house_id=oi_house_id"

        sql_where = [] # 用来保存sql语句的where条件
        sql_params = {} # 用来保存sql查询所需的动态数据

        if start_date and end_date:
            sql_part = "((oi_begin_date>%(end_date)s or oi_end_date<%(start_date)s) " \
                       "or (oi_begin_date is null and oi_end_date is null))"
            sql_where.append(sql_part)
            sql_params["start_date"] = start_date
            sql_params["end_date"] = end_date
        elif start_date:
            sql_part = "(oi_end_date<%(start_date)s or (oi_begin_date is null and oi_end_date is null))"
            sql_where.append(sql_part)
            sql_params["start_date"] = start_date
        elif end_date:
            sql_part = "(oi_begin_date>%(end_date)s or (oi_begin_date is null and oi_end_date is null))"
            sql_where.append(sql_part)
            sql_params["end_date"] = end_date

        if area_id:
            sql_part = "hi_area_id=%(area_id)s"
            sql_where.append(sql_part)
            sql_params["area_id"] = area_id

        if sql_where:
            sql += " where "
            sql += " and ".join(sql_where)

        # 有了where条件，先查询总条目数
        try:
            ret = self.db.get(sql_total_count, **sql_params)
        except Exception as e:
            logging.error(e)
            total_page = -1
        else:
            total_page = int(math.ceil(ret["count"] / float(constants.HOUSE_LIST_PAGE_CAPACITY)))
            page = int(page)
            if page>total_page:
                return self.write(dict(errcode=RET.OK, errmsg="OK", data=[], total_page=total_page))

        # 排序
        if "new" == sort_key: # 按最新上传时间排序
            sql += " order by hi_ctime desc"
        elif "booking" == sort_key: # 最受欢迎
            sql += " order by hi_order_count desc"
        elif "price-inc" == sort_key: # 价格由低到高
            sql += " order by hi_price asc"
        elif "price-des" == sort_key: # 价格由高到低
            sql += " order by hi_price desc"

        # 分页
        # limit 10 返回前10条
        # limit 20,3 从20条开始，返回3条数据
        if 1 == page:
            sql += " limit %s" % constants.HOUSE_LIST_PAGE_CAPACITY
        else:
            sql += " limit %s,%s" % ((page-1)*constants.HOUSE_LIST_PAGE_CAPACITY, constants.HOUSE_LIST_PAGE_CAPACITY)

        logging.debug(sql)
        try:
            ret = self.db.query(sql, **sql_params)
        except Exception as e:
            logging.error(e)
            return self.write(dict(errcode=RET.DBERR, errmsg="查询出错"))
        data = []
        if ret:
            for l in ret:
                house = dict(
                    house_id=l["hi_house_id"],
                    title=l["hi_title"],
                    price=l["hi_price"],
                    room_count=l["hi_room_count"],
                    address=l["hi_address"],
                    order_count=l["hi_order_count"],
                    avatar=constants.QINIU_URL_PREFIX+l["up_avatar"] if l.get("up_avatar") else "",
                    image_url=constants.QINIU_URL_PREFIX+l["hi_index_image_url"] if l.get("hi_index_image_url") else ""
                )
                data.append(house)
        self.write(dict(errcode=RET.OK, errmsg="OK", data=data, total_page=total_page))


class HouseListRedisHandler(BaseHandler):
    """使用了缓存的房源列表页面"""
    def get(self):
        """get方式用来获取数据库数据，本身的逻辑不会对数据库数据产生影响"""
        """
        传入参数说明
        start_date 用户查询的起始时间 sd     非必传   ""          "2017-02-28"
        end_date    用户查询的终止时间 ed    非必传   ""
        area_id     用户查询的区域条件   aid 非必传   ""
        sort_key    排序的关键词     sk     非必传   "new"      "new" "booking" "price-inc"  "price-des"
        page        返回的数据页数     p     非必传   1
        """
        # 获取参数
        start_date = self.get_argument("sd", "")
        end_date = self.get_argument("ed", "")
        area_id = self.get_argument("aid", "")
        sort_key = self.get_argument("sk", "new")
        page = self.get_argument("p", "1")

        # 检查参数
        # 判断日期格式、sort_Key 字段的值、page的整数

        # 先从redis中获取数据
        try:
            redis_key = "houses_%s_%s_%s_%s" % (start_date, end_date, area_id, sort_key)
            ret = self.redis.hget(redis_key, page)
        except Exception as e:
            logging.error(e)
            ret = None
        if ret:
            logging.info("hit redis")
            return self.write(ret)


        # 数据查询
        # 涉及到表： ih_house_info 房屋的基本信息  ih_user_profile 房东的用户信息 ih_order_info 房屋订单数据

        sql = "select distinct hi_title,hi_house_id,hi_price,hi_room_count,hi_address,hi_order_count,up_avatar,hi_index_image_url,hi_ctime" \
              " from ih_house_info inner join ih_user_profile on hi_user_id=up_user_id left join ih_order_info" \
              " on hi_house_id=oi_house_id"

        sql_total_count = "select count(distinct hi_house_id) count from ih_house_info inner join ih_user_profile on hi_user_id=up_user_id " \
                          "left join ih_order_info on hi_house_id=oi_house_id"

        sql_where = [] # 用来保存sql语句的where条件
        sql_params = {} # 用来保存sql查询所需的动态数据

        if start_date and end_date:
            sql_part = "((oi_begin_date>%(end_date)s or oi_end_date<%(start_date)s) " \
                       "or (oi_begin_date is null and oi_end_date is null))"
            sql_where.append(sql_part)
            sql_params["start_date"] = start_date
            sql_params["end_date"] = end_date
        elif start_date:
            sql_part = "(oi_end_date<%(start_date)s or (oi_begin_date is null and oi_end_date is null))"
            sql_where.append(sql_part)
            sql_params["start_date"] = start_date
        elif end_date:
            sql_part = "(oi_begin_date>%(end_date)s or (oi_begin_date is null and oi_end_date is null))"
            sql_where.append(sql_part)
            sql_params["end_date"] = end_date

        if area_id:
            sql_part = "hi_area_id=%(area_id)s"
            sql_where.append(sql_part)
            sql_params["area_id"] = area_id

        if sql_where:
            sql += " where "
            sql += " and ".join(sql_where)

        # 有了where条件，先查询总条目数
        try:
            ret = self.db.get(sql_total_count, **sql_params)
        except Exception as e:
            logging.error(e)
            total_page = -1
        else:
            total_page = int(math.ceil(ret["count"] / float(constants.HOUSE_LIST_PAGE_CAPACITY)))
            page = int(page)
            if page>total_page:
                return self.write(dict(errcode=RET.OK, errmsg="OK", data=[], total_page=total_page))

        # 排序
        if "new" == sort_key: # 按最新上传时间排序
            sql += " order by hi_ctime desc"
        elif "booking" == sort_key: # 最受欢迎
            sql += " order by hi_order_count desc"
        elif "price-inc" == sort_key: # 价格由低到高
            sql += " order by hi_price asc"
        elif "price-des" == sort_key: # 价格由高到低
            sql += " order by hi_price desc"

        # 分页
        # limit 10 返回前10条
        # limit 20,3 从20条开始，返回3条数据
        if 1 == page:
            sql += " limit %s" % (constants.HOUSE_LIST_PAGE_CAPACITY * constants.HOUSE_LIST_PAGE_CACHE_NUM)
        else:
            sql += " limit %s,%s" % ((page-1)*constants.HOUSE_LIST_PAGE_CAPACITY, constants.HOUSE_LIST_PAGE_CAPACITY * constants.HOUSE_LIST_PAGE_CACHE_NUM)

        logging.debug(sql)
        try:
            ret = self.db.query(sql, **sql_params)
        except Exception as e:
            logging.error(e)
            return self.write(dict(errcode=RET.DBERR, errmsg="查询出错"))
        data = []
        if ret:
            for l in ret:
                house = dict(
                    house_id=l["hi_house_id"],
                    title=l["hi_title"],
                    price=l["hi_price"],
                    room_count=l["hi_room_count"],
                    address=l["hi_address"],
                    order_count=l["hi_order_count"],
                    avatar=constants.QINIU_URL_PREFIX+l["up_avatar"] if l.get("up_avatar") else "",
                    image_url=constants.QINIU_URL_PREFIX+l["hi_index_image_url"] if l.get("hi_index_image_url") else ""
                )
                data.append(house)

        # 对与返回的多页面数据进行分页处理
        # 首先取出用户想要获取的page页的数据
        current_page_data = data[:constants.HOUSE_LIST_PAGE_CAPACITY]
        house_data = {}
        house_data[page] = json.dumps(dict(errcode=RET.OK, errmsg="OK", data=current_page_data, total_page=total_page))
        # 将多取出来的数据分页
        i = 1
        while 1:
            page_data = data[i*constants.HOUSE_LIST_PAGE_CAPACITY: (i+1)*constants.HOUSE_LIST_PAGE_CAPACITY]
            if not page_data:
                break
            house_data[page+i] = json.dumps(dict(errcode=RET.OK, errmsg="OK", data=page_data, total_page=total_page))
            i += 1
        try:
            redis_key = "houses_%s_%s_%s_%s" % (start_date, end_date, area_id, sort_key)
            self.redis.hmset(redis_key, house_data)
            self.redis.expire(redis_key, constants.REDIS_HOUSE_LIST_EXPIRES_SECONDS)
        except Exception as e:
            logging.error(e)

        self.write(house_data[page])

```

# 21，编写个人中心api，Profile.py文件。
```
# coding:utf-8

import logging
import constants

from handlers.BaseHandler import BaseHandler
from utils.response_code import RET
from utils.qiniu_storage import storage
from utils.commons import required_login


class AvatarHandler(BaseHandler):
    """上传头像"""
    @required_login
    def post(self):
        files = self.request.files.get("avatar")
        if not files:
            return self.write(dict(errcode=RET.PARAMERR, errmsg="未传图片"))
        avatar = files[0]["body"]
        # 调用七牛上传图片
        try:
            file_name = storage(avatar)
        except Exception as e:
            logging.error(e)
            return self.write(dict(errcode=RET.THIRDERR, errmsg="上传失败"))

        # 从session数据中取出user_id
        user_id = self.session.data["user_id"]

        # 保存图片名（即图片url）到数据中
        sql = "update ih_user_profile set up_avatar=%(avatar)s where up_user_id=%(user_id)s"
        try:
            row_count = self.db.execute_rowcount(sql, avatar=file_name, user_id=user_id)
        except Exception as e:
            logging.error(e)
            return self.write(dict(errcode=RET.DBERR, errmsg="保存错误"))
        self.write(dict(errcode=RET.OK, errmsg="保存成功", data="%s%s" % (constants.QINIU_URL_PREFIX, file_name)))


class ProfileHandler(BaseHandler):
    """个人信息"""
    @required_login
    def get(self):
        user_id = self.session.data['user_id']
        try:
            ret = self.db.get("select up_name,up_mobile,up_avatar from ih_user_profile where up_user_id=%s", user_id)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"get data error"})
        if ret["up_avatar"]:
            img_url = constants.QINIU_URL_PREFIX + ret["up_avatar"]
        else:
            img_url = None
        self.write({"errcode":RET.OK, "errmsg":"OK",
                    "data":{"user_id":user_id, "name":ret["up_name"], "mobile":ret["up_mobile"], "avatar":img_url}})


class NameHandler(BaseHandler):
    """用户名"""
    @required_login
    def post(self):
        # 从session中获取用户身份,user_id
        user_id = self.session.data["user_id"]

        # 获取用户想要设置的用户名
        name = self.json_args.get("name")

        # 判断name是否传了，并且不应为空字符串
        # if name == None or "" == name:
        if name in (None, ""):
            return self.write({"errcode":RET.PARAMERR, "errmsg":"params error"})

        # 保存用户昵称name，并同时判断name是否重复（利用数据库的唯一索引)
        try:
            self.db.execute_rowcount("update ih_user_profile set up_name=%s where up_user_id=%s", name, user_id)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"name has exist"})

        # 修改session数据中的name字段，并保存到redis中
        self.session.data["name"] = name
        try:
            self.session.save()
        except Exception as e:
            logging.error(e)
        self.write({"errcode":RET.OK, "errmsg":"OK"})


class AuthHandler(BaseHandler):
    """实名认证"""
    @required_login
    def get(self):
        # 在session中获取用户user_id
        user_id = self.session.data["user_id"]

        # 在数据库中查询信息
        try:
            ret = self.db.get("select up_real_name,up_id_card from ih_user_profile where up_user_id=%s", user_id)
        except Exception as e:
            # 数据库查询出错
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"get data failed"})
        logging.debug(ret)
        if not ret:
            return self.write({"errcode":RET.NODATA, "errmsg":"no data"})
        self.write({"errcode":RET.OK, "errmsg":"OK", "data":{"real_name":ret.get("up_real_name", ""), "id_card":ret.get("up_id_card", "")}})

    @required_login
    def post(self):
        user_id = self.session.data["user_id"]
        real_name = self.json_args.get("real_name")
        id_card = self.json_args.get("id_card")
        if real_name in (None, "") or id_card in (None, ""):
            return self.write({"errcode":RET.PARAMERR, "errmsg":"params error"})
        # 判断身份证号格式
        try:
            self.db.execute_rowcount("update ih_user_profile set up_real_name=%s,up_id_card=%s where up_user_id=%s", real_name, id_card, user_id)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"update failed"})
        self.write({"errcode":RET.OK, "errmsg":"OK"})
```
将urls.py中的相应api反注释掉。

# 21，编写html/auth.html对应的auth.js。
```
function showSuccessMsg() {
    $('.popup_con').fadeIn('fast', function() {
        setTimeout(function(){
            $('.popup_con').fadeOut('fast',function(){}); 
        },1000) 
    });
}

function getCookie(name) {
    var r = document.cookie.match("\\b" + name + "=([^;]*)\\b");
    return r ? r[1] : undefined;
}

$(document).ready(function(){
    $.get("/api/profile/auth", function(data){
        if ("4101" == data.errcode) {
            location.href = "/login.html";
        }
        else if ("0" == data.errcode) {
            if (data.data.real_name && data.data.id_card) {
                $("#real-name").val(data.data.real_name);
                $("#id-card").val(data.data.id_card);
                $("#real-name").prop("disabled", true);
                $("#id-card").prop("disabled", true);
                $("#form-auth>input[type=submit]").hide();
            }
        }
    }, "json");
    $("#form-auth").submit(function(e){
        e.preventDefault();
        if ($("#real-name").val()=="" || $("#id-card").val() == "") {
            $(".error-msg").show();
        }
        var data = {};
        $(this).serializeArray().map(function(x){data[x.name] = x.value;});
        var jsonData = JSON.stringify(data);
        $.ajax({
            url:"/api/profile/auth",
            type:"POST",
            data: jsonData,
            contentType: "application/json",
            dataType: "json",
            headers:{
                "X-XSRFTOKEN":getCookie("_xsrf"),
            },
            success: function (data) {
                if ("0" == data.errcode) {
                    $(".error-msg").hide();
                    showSuccessMsg();
                    $("#real-name").prop("disabled", true);
                    $("#id-card").prop("disabled", true);
                    $("#form-auth>input[type=submit]").hide();
                }
            }
        });
    })

})

```

# 22，编写首页index.html对应的index.js。
```
//模态框居中的控制
function centerModals(){
    $('.modal').each(function(i){   //遍历每一个模态框
        var $clone = $(this).clone().css('display', 'block').appendTo('body');    
        var top = Math.round(($clone.height() - $clone.find('.modal-content').height()) / 2);
        top = top > 0 ? top : 0;
        $clone.remove();
        $(this).find('.modal-content').css("margin-top", top-30);  //修正原先已经有的30个像素
    });
}

function setStartDate() {
    var startDate = $("#start-date-input").val();
    if (startDate) {
        $(".search-btn").attr("start-date", startDate);
        $("#start-date-btn").html(startDate);
        $("#end-date").datepicker("destroy");
        $("#end-date-btn").html("离开日期");
        $("#end-date-input").val("");
        $(".search-btn").attr("end-date", "");
        $("#end-date").datepicker({
            language: "zh-CN",
            keyboardNavigation: false,
            startDate: startDate,
            format: "yyyy-mm-dd"
        });
        $("#end-date").on("changeDate", function() {
            $("#end-date-input").val(
                $(this).datepicker("getFormattedDate")
            );
        });
        $(".end-date").show();
    }
    $("#start-date-modal").modal("hide");
}

function setEndDate() {
    var endDate = $("#end-date-input").val();
    if (endDate) {
        $(".search-btn").attr("end-date", endDate);
        $("#end-date-btn").html(endDate);
    }
    $("#end-date-modal").modal("hide");
}

function goToSearchPage(th) {
    var url = "/search.html?";
    url += ("aid=" + $(th).attr("area-id"));
    url += "&";
    var areaName = $(th).attr("area-name");
    if (undefined == areaName) areaName="";
    url += ("aname=" + areaName);
    url += "&";
    url += ("sd=" + $(th).attr("start-date"));
    url += "&";
    url += ("ed=" + $(th).attr("end-date"));
    location.href = url;
}

$(document).ready(function(){
    $.get("/api/check_login", function(data) {
        if ("0" == data.errcode) {
            $(".top-bar>.user-info>.user-name").html(data.data.name);
            $(".top-bar>.user-info").show();
        } else {
            $(".top-bar>.register-login").show();
        }
    }, "json");
    $.get("/api/house/index", function(data){
        if ("0" == data.errcode) {
            $(".swiper-wrapper").html(template("swiper-houses-tmpl", {houses:data.houses}));
            $(".area-list").html(template("area-list-tmpl", {areas:data.areas}));
            var mySwiper = new Swiper ('.swiper-container', {
                loop: true,
                autoplay: 2000,
                autoplayDisableOnInteraction: false,
                pagination: '.swiper-pagination',
                paginationClickable: true
            });
            $(".area-list a").click(function(e){
                $("#area-btn").html($(this).html());
                $(".search-btn").attr("area-id", $(this).attr("area-id"));
                $(".search-btn").attr("area-name", $(this).html());
                $("#area-modal").modal("hide");
            });
        }
    });
    $('.modal').on('show.bs.modal', centerModals);      //当模态框出现的时候
    $(window).on('resize', centerModals);               //当窗口大小变化的时候
    $("#start-date").datepicker({
        language: "zh-CN",
        keyboardNavigation: false,
        startDate: "today",
        format: "yyyy-mm-dd"
    });
    $("#start-date").on("changeDate", function() {
        var date = $(this).datepicker("getFormattedDate");
        $("#start-date-input").val(date);
    });
})
```

# 23，编写个人中心profile.html对应的profile.js文件。
```
function showSuccessMsg() {
    $('.popup_con').fadeIn('fast', function() {
        setTimeout(function(){
            $('.popup_con').fadeOut('fast',function(){}); 
        },1000) 
    });
}

function getCookie(name) {
    var r = document.cookie.match("\\b" + name + "=([^;]*)\\b");
    return r ? r[1] : undefined;
}

$(document).ready(function () {
    $.get("/api/profile", function(data){
        if ("4101" == data.errcode) {
            location.href = "/login.html";
        }
        else if ("0" == data.errcode) {
            $("#user-name").val(data.data.name);
            if (data.data.avatar) {
                $("#user-avatar").attr("src", data.data.avatar);
            }
        }
    })
    $("#form-avatar").submit(function (e) {
        // 组织浏览器对于表单的默认行为
        e.preventDefault();
        $('.image_uploading').fadeIn('fast');
        var options = {
            url: "/api/profile/avatar",
            method: "post",
            dataType: "json",
            headers: {
                "X-XSRFTOKEN": getCookie("_xsrf")
            },
            success: function (data) {
                if ("0" == data.errcode) {
                    $('.image_uploading').fadeOut('fast');
                    $("#user-avatar").attr("src", data.data)
                } else if ("4101" == data.errcode) {
                    location.href = "/login.html";
                }
            }
        };
        $(this).ajaxSubmit(options);
    })
    $("#form-name").submit(function(e){
        e.preventDefault();
        var data = {};
        $(this).serializeArray().map(function(x){data[x.name] = x.value;});
        var jsonData = JSON.stringify(data);
        $.ajax({
            url:"/api/profile/name",
            type:"POST",
            data: jsonData,
            contentType: "application/json",
            dataType: "json",
            headers:{
                "X-XSRFTOKEN":getCookie("_xsrf"),
            },
            success: function (data) {
                if ("0" == data.errcode) {
                    $(".error-msg").hide();
                    showSuccessMsg(); // 展示保存成功的页面效果
                } else if ("4001" == data.errcode) {
                    $(".error-msg").show();
                } else if ("4101" == data.errcode) { // 4101代表用户未登录，强制跳转到登录页面
                    location.href = "/login.html";
                }
            }
        });
    })
})

```

# 24，编写my.js。
```
function logout() {
    $.get("/api/logout", function(data){
        if (0 == data.errcode) {
            location.href = "/";
        }
    })
}

$(document).ready(function(){
    $.get("/api/profile", function(data) {
        if ("4101" == data.errcode) {
            location.href = "/login.html";
        }
        else if ("0" == data.errcode) {
            $("#user-name").html(data.data.name);
            $("#user-mobile").html(data.data.mobile);
            if (data.data.avatar) {
                $("#user-avatar").attr("src", data.data.avatar);
            }
        }
    }, "json");
})
```

# 25，编写myhouse.js。
```
$(document).ready(function(){
    $.get("/api/profile/auth", function(data){
        if ("4101" == data.errcode) {
            location.href = "/login.html";
        } else if ("0" == data.errcode) {
            if ("" == data.data.real_name || "" == data.data.id_card || null == data.data.real_name || null == data.data.id_card) {
                $(".auth-warn").show();
                return;
            }
            $.get("/api/house/my", function(result){
                $("#houses-list").html(template("houses-list-tmpl", {houses:result.houses}));
            });
        }
    });
})
```

# 26，编写newhouse.js。
```
function getCookie(name) {
    var r = document.cookie.match("\\b" + name + "=([^;]*)\\b");
    return r ? r[1] : undefined;
}

$(document).ready(function(){
    $.get("/api/house/area", function (data) {
        if ("0" == data.errcode) {
            // html = template("area-tmpl", {areas: data.data});
            // $("#area-id").html(html);
            // console.log(html);
            for (var i=0; i<data.data.length; i++) {
                $("#area-id").append('<option value="'+data.data[i].area_id+'">'+data.data[i].name+'</option>');
            }
        }
    }, "json")

    $("#form-house-info").submit(function(e){
        e.preventDefault();
        var formData = $(this).serializeArray();
        for (var i=0; i<formData.length; i++) {
            if (!formData[i].value) {
                $(".error-msg").show();
                return;
            }
        }
        var data = {};
        $(this).serializeArray().map(function(x){data[x.name] = x.value;});
        var facility = []; // 用来保存勾选了的设施编号
        // 通过jquery筛选出勾选了的页面元素
        // 通过each方法遍历元素
        $("input:checkbox:checked[name=facility]").each(function(i){facility[i] = this.value;});
        data.facility = facility;
        var jsonData = JSON.stringify(data);
        $.ajax({
            url:"/api/house/info",
            type:"POST",
            data: jsonData,
            contentType: "application/json",
            dataType: "json",
            headers:{
                "X-XSRFTOKEN":getCookie("_xsrf"),
            },
            success: function (data) {
                if ("4101" == data.errcode) {
                    location.href = "/login.html";
                } else if ("0" == data.errcode) {
                    $("#house-id").val(data.house_id);
                    $(".error-msg").hide();
                    $("#form-house-info").hide();
                    $("#form-house-image").show();
                }
            }
        });
    })
    $("#form-house-image").submit(function(e){
        e.preventDefault();
        $('.popup_con').fadeIn('fast');
        var options = {
            url:"/api/house/image",
            type:"POST",
            headers:{
                "X-XSRFTOKEN":getCookie("_xsrf"),
            },
            success: function(data){
                if ("4101" == data.errcode) {
                    location.href = "/login.html";
                } else if ("0" == data.errcode) {
                    $(".house-image-cons").append('<img src="'+ data.url+'">');
                    $('.popup_con').fadeOut('fast');
                }
            }
        };
        $(this).ajaxSubmit(options);
    });
})
```

# 27，编写详情页detail.js。
```
function hrefBack() {
    history.go(-1);
}

function decodeQuery(){
    var search = decodeURI(document.location.search);
    return search.replace(/(^\?)/, '').split('&').reduce(function(result, item){
        values = item.split('=');
        result[values[0]] = values[1];
        return result;
    }, {});
}

$(document).ready(function(){
    var house_id = decodeQuery()["id"];
    $.get("/api/house/info?house_id="+house_id, function (data) {
        if ("0" == data.errcode) {
            $(".swiper-container").html(template("house-image-tmpl", {"img_urls":data.data.images, "price":data.data.price}));
            $(".detail-con").html(template("house-detail-tmpl", {"house":data.data}));
            var mySwiper = new Swiper ('.swiper-container', {
                loop: true,
                autoplay: 2000,
                autoplayDisableOnInteraction: false,
                pagination: '.swiper-pagination',
                paginationType: 'fraction'
            })
            // data.user_id为访问页面用户,data.data.user_id为房东
            if (data.user_id != data.data.user_id) {
                $(".book-house").attr("href", "/booking.html?hid="+house_id);
                $(".book-house").show();
            }
        }
    }, "json")
})
```

# 28，编写Orders.py文件。订单api完成了，所有的api就都完成了。
```
# coding:utf-8

import logging
import datetime
import constants

from handlers.BaseHandler import BaseHandler
from utils.commons import required_login
from utils.response_code import RET


class OrderHandler(BaseHandler):
    """订单"""
    @required_login
    def post(self):
        """提交订单"""
        user_id = self.session.data["user_id"]
        house_id = self.json_args.get("house_id")
        start_date = self.json_args.get("start_date")
        end_date = self.json_args.get("end_date")
        # 参数检查
        if not all((house_id, start_date, end_date)):
            return self.write({"errcode":RET.PARAMERR, "errmsg":"params error"})
        # 查询房屋是否存在
        try:
            house = self.db.get("select hi_price,hi_user_id from ih_house_info where hi_house_id=%s", house_id)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"get house error"})
        if not house:
            return self.write({"errcode":RET.NODATA, "errmsg":"no data"})
        # 预订的房屋是否是房东自己的
        if user_id == house["hi_user_id"]:
            return self.write({"errcode":RET.ROLEERR, "errmsg":"user is forbidden"})
        # 判断日期是否可以
        # start_date 与 end_date 两个参数是字符串，需要转为datetime类型进行比较
        # 比较start_date 是否 比 end_date小
        days = (datetime.datetime.strptime(end_date, "%Y-%m-%d") - datetime.datetime.strptime(start_date, "%Y-%m-%d")).days + 1
        if days<=0:
            return self.write({"errcode":RET.PARAMERR, "errmsg":"date params error"})
        # 确保用户预订的时间内，房屋没有被别人下单
        try:
            ret = self.db.get("select count(*) counts from ih_order_info where oi_house_id=%(house_id)s "
                              "and oi_begin_date<%(end_date)s and oi_end_date>%(start_date)s",
                              house_id=house_id, end_date=end_date, start_date=start_date)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"get date error"})
        if ret["counts"] > 0:
            return self.write({"errcode":RET.DATAERR, "errmsg":"serve date error"})
        amount = days * house["hi_price"]
        try:
            # 保存订单数据ih_order_info，
            self.db.execute("insert into ih_order_info(oi_user_id,oi_house_id,oi_begin_date,oi_end_date,oi_days,oi_house_price,oi_amount)"
                            " values(%(user_id)s,%(house_id)s,%(begin_date)s,%(end_date)s,%(days)s,%(price)s,%(amount)s);"
                            "update ih_house_info set hi_order_count=hi_order_count+1 where hi_house_id=%(house_id)s;",
                            user_id=user_id, house_id=house_id, begin_date=start_date, end_date=end_date, days=days, price=house["hi_price"], amount=amount)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"save data error"})
        self.write({"errcode":RET.OK, "errmsg":"OK"})


class MyOrdersHandler(BaseHandler):
    """我的订单"""
    @required_login
    def get(self):
        user_id = self.session.data["user_id"]

        # 用户的身份，用户想要查询作为房客下的单，还是想要查询作为房东 被人下的单
        role = self.get_argument("role", "")
        try:
            # 查询房东订单
            if "landlord" == role:
                ret = self.db.query("select oi_order_id,hi_title,hi_index_image_url,oi_begin_date,oi_end_date,oi_ctime,"
                                    "oi_days,oi_amount,oi_status,oi_comment from ih_order_info inner join ih_house_info "
                                    "on oi_house_id=hi_house_id where hi_user_id=%s order by oi_ctime desc", user_id)
            else:
                ret = self.db.query("select oi_order_id,hi_title,hi_index_image_url,oi_begin_date,oi_end_date,oi_ctime,"
                                    "oi_days,oi_amount,oi_status,oi_comment from ih_order_info inner join ih_house_info "
                                    "on oi_house_id=hi_house_id where oi_user_id=%s order by oi_ctime desc", user_id)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"get data error"})
        orders = []
        if ret:
            for l in ret:
                order = {
                    "order_id":l["oi_order_id"],
                    "title":l["hi_title"],
                    "img_url":constants.QINIU_URL_PREFIX + l["hi_index_image_url"] if l["hi_index_image_url"] else "",
                    "start_date":l["oi_begin_date"].strftime("%Y-%m-%d"),
                    "end_date":l["oi_end_date"].strftime("%Y-%m-%d"),
                    "ctime":l["oi_ctime"].strftime("%Y-%m-%d"),
                    "days":l["oi_days"],
                    "amount":l["oi_amount"],
                    "status":l["oi_status"],
                    "comment":l["oi_comment"] if l["oi_comment"] else ""
                }
                orders.append(order)
        self.write({"errcode":RET.OK, "errmsg":"OK", "orders":orders})


class AcceptOrderHandler(BaseHandler):
    """接单"""
    @required_login
    def post(self):
        # 处理的订单编号
        order_id = self.json_args.get("order_id")
        user_id = self.session.data["user_id"]
        if not order_id:
            return self.write({"errcode":RET.PARAMERR, "errmsg":"params error"})

        try:
            # 确保房东只能修改属于自己房子的订单
            self.db.execute("update ih_order_info set oi_status=3 where oi_order_id=%(order_id)s and oi_house_id in "
                            "(select hi_house_id from ih_house_info where hi_user_id=%(user_id)s) and oi_status=0",
                            # update ih_order_info inner join ih_house_info on oi_house_id=hi_house_id set oi_status=3 where
                            # oi_order_id=%(order_id)s and hi_user_id=%(user_id)s
                            order_id=order_id, user_id=user_id)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"DB error"})
        self.write({"errcode":RET.OK, "errmsg":"OK"})


class RejectOrderHandler(BaseHandler):
    """拒单"""
    @required_login
    def post(self):
        user_id = self.session.data["user_id"]
        order_id = self.json_args.get("order_id")
        reject_reason = self.json_args.get("reject_reason")
        if not all((order_id, reject_reason)):
            return self.write({"errcode":RET.PARAMERR, "errmsg":"params error"})
        try:
            self.db.execute("update ih_order_info set oi_status=6,oi_comment=%(reject_reason)s "
                            "where oi_order_id=%(order_id)s and oi_house_id in (select hi_house_id from ih_house_info "
                            "where hi_user_id=%(user_id)s) and oi_status=0",
                            reject_reason=reject_reason, order_id=order_id, user_id=user_id)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"DB error"})
        self.write({"errcode":RET.OK, "errmsg":"OK"})


class OrderCommentHandler(BaseHandler):
    """评论"""
    @required_login
    def post(self):
        user_id = self.session.data["user_id"]
        order_id = self.json_args.get("order_id")
        comment = self.json_args.get("comment")
        if not all((order_id, comment)):
            return self.write({"errcode":RET.PARAMERR, "errmsg":"params error"})
        try:
            # 需要确保只能评论自己下的订单
            self.db.execute("update ih_order_info set oi_status=4,oi_comment=%(comment)s where oi_order_id=%(order_id)s "
                            "and oi_status=3 and oi_user_id=%(user_id)s", comment=comment, order_id=order_id, user_id=user_id)
        except Exception as e:
            logging.error(e)
            return self.write({"errcode":RET.DBERR, "errmsg":"DB error"})

        # 同步更新redis缓存中关于该房屋的评论信息，此处的策略是直接删除redis缓存中的该房屋数据
        try:
            ret = self.db.get("select oi_house_id from ih_order_info where oi_order_id=%s", order_id)
            if ret:
                self.redis.delete("house_info_%s" % ret["oi_house_id"])
        except Exception as e:
            logging.error(e)
        self.write({"errcode":RET.OK, "errmsg":"OK"})
```

# 29，编写orders.js。
```
//模态框居中的控制
function centerModals(){
    $('.modal').each(function(i){   //遍历每一个模态框
        var $clone = $(this).clone().css('display', 'block').appendTo('body');    
        var top = Math.round(($clone.height() - $clone.find('.modal-content').height()) / 2);
        top = top > 0 ? top : 0;
        $clone.remove();
        $(this).find('.modal-content').css("margin-top", top-30);  //修正原先已经有的30个像素
    });
}

function getCookie(name) {
    var r = document.cookie.match("\\b" + name + "=([^;]*)\\b");
    return r ? r[1] : undefined;
}

$(document).ready(function(){
    $('.modal').on('show.bs.modal', centerModals);      //当模态框出现的时候
    $(window).on('resize', centerModals);
    $.get("/api/order/my?role=custom", function(data){
        if ("0" == data.errcode) {
            $(".orders-list").html(template("orders-list-tmpl", {orders:data.orders}));
            $(".order-comment").on("click", function(){
                var orderId = $(this).parents("li").attr("order-id");
                $(".modal-comment").attr("order-id", orderId);
            });
            $(".modal-comment").on("click", function(){
                var orderId = $(this).attr("order-id");
                var comment = $("#comment").val()
                if (!comment) return;
                var data = {
                    order_id:orderId,
                    comment:comment
                };
                $.ajax({
                    url:"/api/order/comment",
                    type:"POST",
                    data:JSON.stringify(data),
                    contentType:"application/json",
                    dataType:"json",
                    headers:{
                        "X-XSRFTOKEN":getCookie("_xsrf"),
                    },
                    success:function (data) {
                        if ("4101" == data.errcode) {
                            location.href = "/login.html";
                        } else if ("0" == data.errcode) {
                            $(".orders-list>li[order-id="+ orderId +"]>div.order-content>div.order-text>ul li:eq(4)>span").html("已完成");
                            $(".order-operate").hide();
                            $("#comment-modal").modal("hide");
                        }
                    }
                });
            });
        }
    });
});
```

30，编写booking.js文件，实现下订单功能。
```
function hrefBack() {
    history.go(-1);
}

function getCookie(name) {
    var r = document.cookie.match("\\b" + name + "=([^;]*)\\b");
    return r ? r[1] : undefined;
}

function decodeQuery(){
    var search = decodeURI(document.location.search);
    return search.replace(/(^\?)/, '').split('&').reduce(function(result, item){
        values = item.split('=');
        result[values[0]] = values[1];
        return result;
    }, {});
}

function showErrorMsg(msg) {
    $(".popup>p").html(msg);
    $('.popup_con').fadeIn('fast', function() {
        setTimeout(function(){
            $('.popup_con').fadeOut('fast',function(){}); 
        },1000) 
    });
}

$(document).ready(function(){
    $.get("/api/check_login", function(data) {
        if ("0" != data.errcode) {
            location.href = "/login.html";
        }
    }, "json");
    $(".input-daterange").datepicker({
        format: "yyyy-mm-dd",
        startDate: "today",
        language: "zh-CN",
        autoclose: true
    });
    $(".input-daterange").on("changeDate", function(){
        var startDate = $("#start-date").val();
        var endDate = $("#end-date").val();

        if (startDate && endDate && startDate > endDate) {
            showErrorMsg("日期有误，请重新选择！");
        } else {
            var sd = new Date(startDate);
            var ed = new Date(endDate);
            days = (ed - sd)/(1000*3600*24) + 1;
            var price = $(".house-text>p>span").html();
            var amount = days * parseFloat(price);
            $(".order-amount>span").html(amount.toFixed(2) + "(共"+ days +"晚)");
        }
    });
    var queryData = decodeQuery();
    var houseId = queryData["hid"];
    $.get("/api/house/info?house_id=" + houseId, function(data){
        if ("0" == data.errcode) {
            $(".house-info>img").attr("src", data.data.images[0]);
            $(".house-text>h3").html(data.data.title);
            $(".house-text>p>span").html((data.data.price/100.0).toFixed(0));
        }
    });
    $(".submit-btn").on("click", function(e) {
        if ($(".order-amount>span").html()) {
            $(this).prop("disabled", true);
            var startDate = $("#start-date").val();
            var endDate = $("#end-date").val();
            var data = {
                "house_id":houseId,
                "start_date":startDate,
                "end_date":endDate
            };
            $.ajax({
                url:"/api/order",
                type:"POST",
                data: JSON.stringify(data), 
                contentType: "application/json",
                dataType: "json",
                headers:{
                    "X-XSRFTOKEN":getCookie("_xsrf"),
                },
                success: function (data) {
                    if ("4101" == data.errcode) {
                        location.href = "/login.html";
                    } else if ("4004" == data.errcode) {
                        showErrorMsg("房间已被抢定，请重新选择日期！"); 
                    } else if ("0" == data.errcode) {
                        location.href = "/orders.html";
                    }
                }
            });
        }
    });
})

```

# 31，编写lorders.js。
```
//模态框居中的控制
function centerModals(){
    $('.modal').each(function(i){   //遍历每一个模态框
        var $clone = $(this).clone().css('display', 'block').appendTo('body');    
        var top = Math.round(($clone.height() - $clone.find('.modal-content').height()) / 2);
        top = top > 0 ? top : 0;
        $clone.remove();
        $(this).find('.modal-content').css("margin-top", top-30);  //修正原先已经有的30个像素
    });
}

function getCookie(name) {
    var r = document.cookie.match("\\b" + name + "=([^;]*)\\b");
    return r ? r[1] : undefined;
}

$(document).ready(function(){
    $('.modal').on('show.bs.modal', centerModals);      //当模态框出现的时候
    $(window).on('resize', centerModals);
    $.get("/api/order/my?role=landlord", function(data){
        if ("0" == data.errcode) {
            $(".orders-list").html(template("orders-list-tmpl", {orders:data.orders}));
            $(".order-accept").on("click", function(){
                var orderId = $(this).parents("li").attr("order-id");
                $(".modal-accept").attr("order-id", orderId);
            });
            $(".modal-accept").on("click", function(){
                var orderId = $(this).attr("order-id");
                $.ajax({
                    url:"/api/order/accept",
                    type:"POST",
                    data:'{"order_id":'+ orderId +'}',
                    contentType:"application/json",
                    headers:{
                        "X-XSRFTOKEN":getCookie("_xsrf"),
                    },
                    dataType:"json",
                    success:function (data) {
                        if ("4101" == data.errcode) {
                            location.href = "/login.html";
                        } else if ("0" == data.errcode) {
                            $(".orders-list>li[order-id="+ orderId +"]>div.order-content>div.order-text>ul li:eq(4)>span").html("已接单");
                            $("ul.orders-list>li[order-id="+ orderId +"]>div.order-title>div.order-operate").hide();
                            $("#accept-modal").modal("hide");
                        }
                    }
                })
            });
            $(".order-reject").on("click", function(){
                var orderId = $(this).parents("li").attr("order-id");
                $(".modal-reject").attr("order-id", orderId);
            });
            $(".modal-reject").on("click", function(){
                var orderId = $(this).attr("order-id");
                var reject_reason = $("#reject-reason").val()
                if (!reject_reason) return;
                var data = {
                    order_id:orderId,
                    reject_reason:reject_reason
                };
                $.ajax({
                    url:"/api/order/reject",
                    type:"POST",
                    data:JSON.stringify(data),
                    contentType:"application/json",
                    dataType:"json",
                    headers: {
                        "X-XSRFTOKEN":getCookie("_xsrf"),
                    },
                    success:function (data) {
                        if ("4101" == data.errcode) {
                            location.href = "/login.html";
                        } else if ("0" == data.errcode) {
                            $(".orders-list>li[order-id="+ orderId +"]>div.order-content>div.order-text>ul li:eq(4)>span").html("已拒单");
                            $(".order-operate").hide();
                            $("#reject-modal").modal("hide");
                        }
                    }
                });
            })
        }
    });
});
```

# 32，实现搜索功能，search.js文件。
```
var cur_page = 1;
var next_page = 1;
var total_page = 1;
var house_data_querying = true;

function decodeQuery(){
    var search = decodeURI(document.location.search);
    return search.replace(/(^\?)/, '').split('&').reduce(function(result, item){
        values = item.split('=');
        result[values[0]] = values[1];
        return result;
    }, {});
}

function updateFilterDateDisplay() {
    var startDate = $("#start-date").val();
    var endDate = $("#end-date").val();
    var $filterDateTitle = $(".filter-title-bar>.filter-title").eq(0).children("span").eq(0);
    if (startDate) {
        var text = startDate.substr(5) + "/" + endDate.substr(5);
        $filterDateTitle.html(text);
    } else {
        $filterDateTitle.html("入住日期");
    }
}

function updateHouseData(action="append") {
    var areaId = $(".filter-area>li.active").attr("area-id");
    if (undefined == areaId) areaId = "";
    var startDate = $("#start-date").val();
    var endDate = $("#end-date").val();
    var sortKey = $(".filter-sort>li.active").attr("sort-key");
    var params = {
        aid:areaId,
        sd:startDate,
        ed:endDate,
        sk:sortKey,
        p:next_page
    };
    $.get("/api/house/list2", params, function(data){
        house_data_querying = false;
        if ("0" == data.errcode) {
            if (0 == data.total_page) {
                $(".house-list").html("暂时没有符合您查询的房屋信息。");
            } else {
                total_page = data.total_page;
                if ("append" == action) {
                    cur_page = next_page;
                    $(".house-list").append(template("house-list-tmpl", {houses:data.data}));
                } else if ("renew" == action) {
                    cur_page = 1;
                    $(".house-list").html(template("house-list-tmpl", {houses:data.data}));
                }
            }
        }
    })
}

$(document).ready(function(){
    var queryData = decodeQuery();
    var startDate = queryData["sd"];
    var endDate = queryData["ed"];
    $("#start-date").val(startDate);
    $("#end-date").val(endDate);
    updateFilterDateDisplay();
    var areaName = queryData["aname"];
    if (!areaName) areaName = "位置区域";
    $(".filter-title-bar>.filter-title").eq(1).children("span").eq(0).html(areaName);

    $.get("/api/house/area", function(data){
        if ("0" == data.errcode) {
            var areaId = queryData["aid"];
            if (areaId) {
                for (var i=0; i<data.data.length; i++) {
                    areaId = parseInt(areaId);
                    if (data.data[i].area_id == areaId) {
                        $(".filter-area").append('<li area-id="'+ data.data[i].area_id+'" class="active">'+ data.data[i].name+'</li>');
                    } else {
                        $(".filter-area").append('<li area-id="'+ data.data[i].area_id+'">'+ data.data[i].name+'</li>');
                    }
                }
            } else {
                for (var i=0; i<data.data.length; i++) {
                    $(".filter-area").append('<li area-id="'+ data.data[i].area_id+'">'+ data.data[i].name+'</li>');
                }
            }
            updateHouseData("renew");
            var windowHeight = $(window).height()
            window.onscroll=function(){
                // var a = document.documentElement.scrollTop==0? document.body.clientHeight : document.documentElement.clientHeight;
                var b = document.documentElement.scrollTop==0? document.body.scrollTop : document.documentElement.scrollTop;
                var c = document.documentElement.scrollTop==0? document.body.scrollHeight : document.documentElement.scrollHeight;
                if(c-b<windowHeight+50){
                    if (!house_data_querying) {
                        house_data_querying = true;
                        if(cur_page < total_page) {
                            next_page = cur_page + 1;
                            updateHouseData();
                        }
                    }
                }
            }
        }
    });


    $(".input-daterange").datepicker({
        format: "yyyy-mm-dd",
        startDate: "today",
        language: "zh-CN",
        autoclose: true
    });
    var $filterItem = $(".filter-item-bar>.filter-item");
    $(".filter-title-bar").on("click", ".filter-title", function(e){
        var index = $(this).index();
        if (!$filterItem.eq(index).hasClass("active")) {
            $(this).children("span").children("i").removeClass("fa-angle-down").addClass("fa-angle-up");
            $(this).siblings(".filter-title").children("span").children("i").removeClass("fa-angle-up").addClass("fa-angle-down");
            $filterItem.eq(index).addClass("active").siblings(".filter-item").removeClass("active");
            $(".display-mask").show();
        } else {
            $(this).children("span").children("i").removeClass("fa-angle-up").addClass("fa-angle-down");
            $filterItem.eq(index).removeClass('active');
            $(".display-mask").hide();
            updateFilterDateDisplay();
            cur_page = 1;
            next_page = 1;
            total_page = 1;
            updateHouseData("renew");
        }
    });
    $(".display-mask").on("click", function(e) {
        $(this).hide();
        $filterItem.removeClass('active');
        updateFilterDateDisplay();
        cur_page = 1;
        next_page = 1;
        total_page = 1;
        updateHouseData("renew");
    });
    $(".filter-item-bar>.filter-area").on("click", "li", function(e) {
        if (!$(this).hasClass("active")) {
            $(this).addClass("active");
            $(this).siblings("li").removeClass("active");
            $(".filter-title-bar>.filter-title").eq(1).children("span").eq(0).html($(this).html());
        } else {
            $(this).removeClass("active");
            $(".filter-title-bar>.filter-title").eq(1).children("span").eq(0).html("位置区域");
        }
    });
    $(".filter-item-bar>.filter-sort").on("click", "li", function(e) {
        if (!$(this).hasClass("active")) {
            $(this).addClass("active");
            $(this).siblings("li").removeClass("active");
            $(".filter-title-bar>.filter-title").eq(2).children("span").eq(0).html($(this).html());
        }
    })
})
```