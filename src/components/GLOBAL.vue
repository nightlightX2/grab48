/* 全局变量定义 Global.vue */
<script>
import axios from "axios";
var GLOBAL = {};
GLOBAL.version = "0.5.0";
GLOBAL.debug = false;

//配置缓存
(GLOBAL.configInit = function() {
  // 默认配置
  GLOBAL.config = {
    isCol: true, //是否收起表格
    isColor: true, //是否开启队伍颜色
    isAutoSync: true, //是否自动更新数据
    version: GLOBAL.version, //版本号
    isShowTip: false, //是否查看过提示
    isAutoLoad: true, //是否自动加载更多
    isReverse: true, //房间是否与app相反的方向显示(最新的在最上)
    smobile: 0,
    spass: null
  };
  // console.log('init')
})();
(GLOBAL.readConfig = function() {
  try {
    /* 尝试获取localStorage中的config项 */
    GLOBAL.config = JSON.parse(localStorage.getItem("config")) || {};
  } catch (e) {
    console.log(e);
  }
  //如果版本号不相同，重置设置。
  if (typeof GLOBAL.config.version === "undefined") {
    GLOBAL.configInit();
    return;
  }
  if (GLOBAL.config.version != GLOBAL.version) {
    GLOBAL.configInit();
  }
  // console.log(GLOBAL.config);
})();
GLOBAL.saveConfig = function() {
  localStorage.setItem("config", JSON.stringify(GLOBAL.config));
};

//配置账户
(GLOBAL.accountInit = function() {
  // 默认配置
  GLOBAL.account = {
    token: "0"
  };
})();
(GLOBAL.accountLoad = function() {
  try {
    /* 尝试获取localStorage中的account项 */
    GLOBAL.account = JSON.parse(localStorage.getItem("account")) || {};
    // 自动登录
  } catch (e) {
    console.log(e);
  }
  //如果没有token，则重置
  if (
    GLOBAL.account.token == "" ||
    typeof GLOBAL.account.token === "undefined"
  ) {
    GLOBAL.accountInit();
  }
})();
GLOBAL.accountSave = function() {
  localStorage.setItem("account", JSON.stringify(GLOBAL.account));
};

//api设置
GLOBAL.api = {
  update: "https://pocketapi.48.cn/user/api/v1/client/update/group_team_star",
  livelist: "https://pocketapi.48.cn/live/api/v1/live/getLiveList",
  openlivelist: "https://pocketapi.48.cn/live/api/v1/live/getOpenLiveList",
  liveone: "https://pocketapi.48.cn/live/api/v1/live/getLiveOne",
  openliveone: "https://pocketapi.48.cn/live/api/v1/live/getOpenLiveOne",

  roomid: "https://pocketapi.48.cn/im/api/v1/im/room/info/type/source",
  roomlio: "https://pocketapi.48.cn/im/api/v1/chatroom/msg/list/homeowner",
  roomlia: "https://pocketapi.48.cn/im/api/v1/chatroom/msg/list/all",

  login: "https://pocketapi.48.cn/user/api/v1/login/app/mobile",
  checkin: "https://pocketapi.48.cn/user/api/v1/checkin",
  userhome: "https://pocketapi.48.cn/user/api/v1/user/info/home"
};

GLOBAL.apiProd = {
  update: "./static/pxy_cache.php?f=update",
  livelist: "./static/pxy_cache.php?f=livelist",
  openlivelist: "./static/pxy_cache.php?f=openlivelist",
  liveone: "./static/pxy_cache.php?f=liveone",
  openliveone: "./static/pxy_cache.php?f=openliveone",

  roomid: "./static/pxy_cache.php?f=roomid",
  roomlio: "./static/pxy_cache.php?f=roomlio",
  roomlia: "./static/pxy_cache.php?f=roomlia",

  login: "./static/pxy_cache.php?f=login",
  checkin: "./static/pxy_cache.php?f=checkin",
  userhome: "./static/pxy_cache.php?f=userhome"
};
GLOBAL.apiDev = {
  update: "./api/user/api/v1/client/update/group_team_star",
  livelist: "./api/live/api/v1/live/getLiveList",
  openlivelist: "./api/live/api/v1/live/getOpenLiveList",
  liveone: "./api/live/api/v1/live/getLiveOne",
  openliveone: "./api/live/api/v1/live/getOpenLiveOne",

  roomid: "./api/im/api/v1/im/room/info/type/source",
  roomlio: "./api/im/api/v1/chatroom/msg/list/homeowner",
  roomlia: "./api/im/api/v1/chatroom/msg/list/all",

  login: "./api/user/api/v1/login/app/mobile",
  checkin: "./api/user/api/v1/checkin",
  userhome: "./api/user/api/v1/user/info/home"
};
if (process.env.NODE_ENV == "development") {
  GLOBAL.api = GLOBAL.apiDev;
} else if (process.env.NODE_ENV == "production") {
  GLOBAL.api = GLOBAL.apiProd;
}

//headers设置
GLOBAL.headers = function(a = false) {
  if (!(
    GLOBAL.account.token == 0 ||
    typeof GLOBAL.account.token === "undefined" ||
    GLOBAL.account.token == null ||
    GLOBAL.account.token == "0"
  )) {
    //加入token
    this.token = GLOBAL.account.token;
  }
  if (a) {
    //加入appInfo
    this.appInfo = `{"vendor":"apple","deviceName":"iPhone 8Plus","deviceId":"123","appVersion":"0.0.1","appBuild":"1","osType":"ios","osVersion":"ios 10.3.3","longitude":1.033,"latitude":1.033}`;
  }
};

//基本信息缓存
GLOBAL.info = {};
GLOBAL.infoLoaded = false;
(GLOBAL.readInfo = function() {
  if (localStorage.getItem("info")) {
    try {
      /* 尝试获取localStorage中的info项 */
      GLOBAL.info = JSON.parse(localStorage.getItem("info"));
      GLOBAL.infoLoaded = true;
    } catch (e) {
      console.log(e);
      GLOBAL.info = {};
    }
  } else {
    GLOBAL.info = {};
  }
})();
GLOBAL.saveInfo = function() {
  localStorage.setItem("info", JSON.stringify(GLOBAL.info));
};

/**
 * 成员id转成员名
 */
GLOBAL.memberId2name = function(userId) {
  // 优化性能
  /*   for (var i in GLOBAL.info.officialInfo) {
    if (GLOBAL.info.officialInfo[i].userId == userId) {
      return GLOBAL.info.officialInfo[i].realName;
    }
  }
  for (var i in GLOBAL.info.starInfo) {
    if (GLOBAL.info.starInfo[i].userId == userId) {
      return GLOBAL.info.starInfo[i].realName;
    }
  } */
  var find = GLOBAL.info.starInfo.find(v => v.userId == userId);
  if (find) {
    return find.realName;
  } else {
    find = GLOBAL.info.officialInfo.find(v => v.userId == userId);
    if (find) {
      return find.realName;
    } else {
      return false;
    }
  }
};
/**
 * 队伍id转队伍名
 */
GLOBAL.teamId2name = function(teamId) {
  for (var i in GLOBAL.info.teamInfo) {
    if (GLOBAL.info.teamInfo[i].teamId == teamId) {
      return GLOBAL.info.teamInfo[i].teamName;
    }
  }
  return false;
};
/**
 * 团体id转团体名
 */
GLOBAL.groupId2name = function(groupId) {
  for (var i in GLOBAL.info.groupInfo) {
    if (GLOBAL.info.groupInfo[i].groupId == groupId) {
      return GLOBAL.info.groupInfo[i].groupName;
    }
  }
  return false;
};
/**
 * 期数id转期数名
 */
GLOBAL.periodId2name = function(periodId) {
  for (var i in GLOBAL.info.periodInfo) {
    if (GLOBAL.info.periodInfo[i].periodId == periodId) {
      return GLOBAL.info.periodInfo[i].periodName;
    }
  }
  return false;
};

/**
 * 成员id转队伍id
 */
GLOBAL.memberId2teamId = function(userId) {
  // 执行过慢
  /*   for (var i in GLOBAL.info.starInfo) {
    if (GLOBAL.info.starInfo[i].userId == userId) {
      return GLOBAL.info.starInfo[i].teamId;
    } */
  var find = GLOBAL.info.starInfo.find(v => v.userId == userId);
  if (find) {
    return find.teamId;
  } else {
    return false;
  }
};

/**
 * 成员id转团体id
 */
GLOBAL.memberId2groupId = function(memberId) {
  for (var i in GLOBAL.info.officialInfo) {
    if (GLOBAL.info.officialInfo[i].userId == memberId) {
      return GLOBAL.info.officialInfo[i].groupId;
    }
  }
  for (var i in GLOBAL.info.starInfo) {
    if (GLOBAL.info.starInfo[i].userId == memberId) {
      return GLOBAL.info.starInfo[i].groupId;
    }
  }
  return false;
};

/**
 * 团体id转首个队伍id
 */
GLOBAL.groupId2firstTeamId = function(groupId) {
  for (var i in GLOBAL.info.teamInfo) {
    if (
      GLOBAL.info.teamInfo[i].teamId != 0 &&
      GLOBAL.info.teamInfo[i].groupId == groupId
    ) {
      return GLOBAL.info.teamInfo[i].teamId;
    }
  }
  return 0;
};

/**
 * 队伍id转颜色
 */
GLOBAL.teamId2color = function(teamId) {
  for (var i in GLOBAL.info.teamInfo) {
    if (GLOBAL.info.teamInfo[i].teamId == teamId) {
      return GLOBAL.info.teamInfo[i].teamColor;
    }
  }
  return false;
};

/**
 * 成员id转颜色
 */
GLOBAL.memberId2color = function(memberId) {
  var teamId = GLOBAL.memberId2teamId(memberId);
  if (teamId == 0) {
    // 如果不存在队伍，则返回其团体的颜色
    return GLOBAL.teamId2color(
      GLOBAL.groupId2firstTeamId(GLOBAL.memberId2groupId(memberId))
    );
  } else {
    // 如果队伍存在，直接返回队伍的颜色
    return GLOBAL.teamId2color(teamId);
  }
};

/**
 * 成员id颜色渲染
 * @returns 颜色代码 '#xxxxxx'
 */
GLOBAL.renderColor = function(memberId) {
  if (GLOBAL.config.isColor) {
    var color = GLOBAL.memberId2color(memberId);
    if (!color) {
      console.log("unexcepted memberId:", memberId);
      return "#000000";
    }
    //避免白色
    if (color.toUpperCase() == "FFFFFF") {
      color = "000000";
    }
    return "#" + color;
  } else {
    return "inherit";
  }
};

/**
 * 团体id颜色渲染
 *
 */
GLOBAL.renderColorG = function(groupId) {
  if (GLOBAL.config.isColor) {
    var teamId = GLOBAL.groupId2firstTeamId(groupId);
    var color = GLOBAL.teamId2color(teamId);
    //避免白色
    if (color.toUpperCase() == "FFFFFF") {
      color = "000000";
    }
    return "#" + color;
  } else {
    return "inherit";
  }
};

/**
 * 统计
 */
GLOBAL.sta = function(name, data) {
  if (_hmt) {
    /* 百度事件统计
    _hmt.push(["_trackEvent", name, action, value]);
    */
    /**
     * action memberId(memberName) / groupId(groupName) / userId(userName)
     * value groupId|teamId|userId|record|next
     */
    var action, value;
    if (name == "liveREQ") {
      if (data.memberId == 0) {
        action = `${data.groupId}(${GLOBAL.groupId2name(data.groupId)})`;
      } else {
        action = `${data.userId}(${GLOBAL.memberId2name(data.userId)})`;
      }
      value = `${data.groupId}(${GLOBAL.groupId2name(data.groupId)}) | ${
        data.teamId
      }(${GLOBAL.teamId2name(data.teamId)}) | ${
        data.userId
      }(${GLOBAL.memberId2name(data.userId)}) | ${data.record} | ${data.next}`;
    }
    if (name == "openREQ") {
      action = `${data.groupId}(${GLOBAL.groupId2name(data.groupId)})`;
      value = `${data.groupId}(${GLOBAL.groupId2name(data.groupId)}) | ${
        data.teamId
      }(${GLOBAL.teamId2name(data.teamId)}) | ${
        data.userId
      }(${GLOBAL.memberId2name(data.userId)}) | ${data.record} | ${data.next}`;
    }
    if (name == "roomREQ") {
      action = `${data.memberId}(${GLOBAL.memberId2name(data.memberId)})`;
      value = `${data.groupId}(${GLOBAL.groupId2name(data.groupId)}) | ${
        data.teamId
      }(${GLOBAL.teamId2name(data.teamId)}) | ${
        data.memberId
      }(${GLOBAL.memberId2name(data.memberId)}) | ${data.nextTime}`;
    }
    if (name == "loginRES") {
      action = `${data.content.userInfo.userId}(${data.content.userInfo.nickname})`;
      value = `${data.content.userInfo.userId}(${data.content.userInfo.nickname}) | ${data.content.userInfo.exp}`;
    }
    //console.log(name, action, value);
    //_hmt.push(['_trackEvent', category, action, opt_label, opt_value]);
    _hmt.push(["_trackEvent", name, action, value]);
  }
};

/**
 * 去重合并
 */
GLOBAL.concatuni = function(arrs = [], id) {
  var temp = {}, //用于id判断重复
    result = [], //最后的新数组
    isU = false; //返回是否有重复
  //去重(唯一id: id)
  arrs.forEach(arr => {
    arr.map((item, index) => {
      if (!temp[item[id]]) {
        result.push(item);
        temp[item[id]] = true;
      } else {
        isU = true;
      }
    });
  });
  return result;
};

GLOBAL.source = "https://source.48.cn"; //图片及资源前缀
GLOBAL.liveUrl = "https://h5.48.cn/2019appshare/liveshare/?id="; //公演直播在线观看前缀
GLOBAL.memberLiveUrl = "https://h5.48.cn/2019appshare/memberLiveShare/?id="; //
GLOBAL.getPicPath = function(picUrl) {
  try {
    return (picUrl.slice(0, 4) == "http" ? "" : GLOBAL.source) + picUrl;
  } catch (e) {
    console.log(picUrl, e);
  }
};

Date.prototype.Format = function(fmt) {
  var o = {
    "M+": this.getMonth() + 1, //月份
    "d+": this.getDate(), //日
    "H+": this.getHours(), //小时
    "m+": this.getMinutes(), //分
    "s+": this.getSeconds(), //秒
    "q+": Math.floor((this.getMonth() + 3) / 3), //季度
    S: this.getMilliseconds() //毫秒
  };
  if (/(y+)/.test(fmt))
    fmt = fmt.replace(
      RegExp.$1,
      (this.getFullYear() + "").substr(4 - RegExp.$1.length)
    );
  for (var k in o)
    if (new RegExp("(" + k + ")").test(fmt))
      fmt = fmt.replace(
        RegExp.$1,
        RegExp.$1.length == 1 ? o[k] : ("00" + o[k]).substr(("" + o[k]).length)
      );
  return fmt;
};
export default GLOBAL;
</script>