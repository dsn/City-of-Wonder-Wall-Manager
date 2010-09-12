// City of Wonder Wall Scanner
// version 0.7 ALPHA!
// 2010-08-27
// Copyright (c) 2010, Dark Side Networks
// Released under the GPL license
// http://www.gnu.org/copyleft/gpl.html
//
// --------------------------------------------------------------------
//
// This is a Greasemonkey user script.  To install it, you need
// Greasemonkey 0.8 or later: http://greasemonkey.mozdev.org/
// Then restart Firefox and revisit this script.
// Under Tools, there will be a new menu item to "Install User Script".
// Accept the default configuration and install.
//
// To uninstall, go to Tools/Manage User Scripts,
// select "City of War Wall Manager", and click Uninstall.
//
// --------------------------------------------------------------------
//
// ==UserScript==
// @name          City of Wonder Wall Manager
// @namespace     cowwm
// @description   Collect Bonuses and Assist Build Marvels for the Facebook game City of Wonder
// @include       http://www.facebook.com/home.php?filter=app_114335335255741*
// @exclude		  http://www.facebook.com/reqs.php*
// @exclude		  http://www.facebook.com/profile.php*
// ==/UserScript==
// CHANGELOG
// 0.7B
// Hide Posts Already Processed Option
// Reset User DB Option
// Updated Status Reports
// UI Updates
// Removed GM Script Options (Moved to Options Panel)
// 0.6A
// Enable "Like" Bonus Option
// Modify Metadata to prevent loading on random FB Pages
// 0.5A
// Enable Multiple User Support
// Options Enabled

// Script Settings
var version = "0.6 Alpha";
var debug = true; // Set to false to disable logging
var logLevel = "info"; // set to debug, info, warn, or error to filter events
// Script Variables
var applicationID = "114335335255741";
var gameTitle = "City of Wonder";
var bonusRegex = "Pick Up Silver|Collect Silver|Collect Bonus";
var marvelRegex = "Help to Collect Bonus";
var legengRegex = "Send Legend to help";
var townRegex = "Visit your city"
// Aliases
var newLine = "<br>&nbsp;&nbsp;";
var indent = "&nbsp;&nbsp;";

function $() {
    if (arguments.length == 1) {
        return document.getElementById(arguments[0]);
    }
    var result = [],
        i = 0,
        el;
    while (el = document.getElementById(arguments[i++])) {
        result.push(el);
    }
    return result;
}
function getDOC(url, callback) {
    GM_xmlhttpRequest({
        method: 'GET',
        url: url,
        onload: function (responseDetails) {
            var dt = document.implementation.createDocumentType("html", "-//W3C//DTD HTML 4.01 Transitional//EN", "http://www.w3.org/TR/html4/loose.dtd"),
                doc = document.implementation.createDocument('', '', dt),
                html = doc.createElement('html');
            html.innerHTML = responseDetails.responseText;
            doc.appendChild(html);
            callback(doc, url);
        }
    });
}


var wonder = {
    core: {
        init: function () {
            if (typeof unsafeWindow != "undefined" && unsafeWindow != unsafeWindow.top) {
                return false;
            }
			wonder.opts.set("worker.status", "");
			wonder.core.uname = document.getElementsByClassName("fbxWelcomeBoxName")[0].text;
			wonder.core.uid = document.getElementsByClassName('fbxWelcomeBoxName')[0].href.split("=")[1];
			wonder.utils.logger("Initializing City of Wonder Wall Manager", "info");
			desc = "<div id='cowwm-totals'><h4 id='cowwm-totals-title'>Total Collection Summary</h4><label for='cowwm-totals-marvels'/>Marvels Assisted: <span id='cowwm-totals-marvels'></span><br/><label for='cowwm-totals-bonus'/>Bonuses Collected: <span id='cowwm-totals-bonus'></span><br/><label for='cowwm-totals-silver' title='Estimated Silver Collected'/>Silver Gained: <span id='cowwm-totals-silver'></span></div><br/><div id='cowwm-summary'><br/></div>" + "<img src='http://photos-b.ak.fbcdn.net/photos-ak-ash1/v27562/61/114335335255741/app_2_114335335255741_9738.gif' id='cowwm-logo' title='Click to open or close Options'/>" + indent + "<span id='cowwm-panel-name' title='Click to open Options'>[CoW Scanner]</span>" + indent + "<a href='javascript:void(0);' id='collectBtn'>Scan Now</a>" + indent + "<span id='cowwm-status'>Loading (Waiting for Facebook to Finish Loading)</span>";
            div = document.createElement('div');
            div.id = 'cowwm-cpanel';
            div.innerHTML = desc;
            document.body.style.paddingBottom = "4em";
            document.body.appendChild(div);
            wonder.utils.logger("Loading Control Panel", "info");
            var autoExpand = wonder.opts.get("opts.autoexpand")
            var autoLike = wonder.opts.get("opts.likeassist")
            var bonusLike = wonder.opts.get("opts.likebonus")
            var dimCompleted = wonder.opts.get("opts.dimcompleted");
            if (autoExpand === undefined) autoExpand = true;
            if (autoLike === undefined) autoLike = true;
            if (dimCompleted === undefined) dimCompleted = true;
            if (autoExpand === true) {
                wonder.opts.autoexpand = true;
                wonder.opts.set("opts.autoexpand", true);
                window.setInterval(wonder.actions.expand, 1000);
            }
            else {
                wonder.opts.autoexpand = false;
                wonder.opts.get("opts.autoexpand", false);
            }
            if (autoLike === true) {
                wonder.opts.autolike = true;
                wonder.opts.set("opts.likeassist", true);
            }
            else {
                wonder.opts.autolike = false;
                wonder.opts.set("opts.likeassist", false);
            }

			 if (bonusLike === true) {
                wonder.opts.likebonus = true;
                wonder.opts.set("opts.likebonus", true);
            }
            else {
                wonder.opts.likebonus = false;
                wonder.opts.set("opts.likebonus", false);
            }

			(dimCompleted === true) ? wonder.opts.dimcompleted = true && wonder.opts.set("opts.dimcompleted", true) : wonder.opts.dimcompleted = false && wonder.opts.set("opts.dimcompleted", false);
			wonder.utils.logger("------------------------------------", "debug");
			wonder.utils.logger("City of Wonder Wall Manager Settings", "debug")
			wonder.utils.logger("-------------------------------------", "debug");
			wonder.utils.logger("User: " + wonder.core.uname + " (" + wonder.core.uid + ")", "debug");
			wonder.utils.logger("Version: " + version, "debug"); 
			wonder.utils.logger("-----------------------------", "debug");
            wonder.utils.logger("Auto Expand More Posts: " + wonder.opts.autoexpand, "debug");
			wonder.utils.logger("Like Bonuses Collected: " + wonder.opts.bonuslike, "debug");
            wonder.utils.logger("Like Marvel Assissted: " + wonder.opts.autolike, "debug");
            wonder.utils.logger("Hide Processed Posts: " + wonder.opts.dimcompleted, "debug");
			wonder.utils.logger("-----------------------------", "debug");
			wonder.reports.totals(wonder.core.uid);
            return true;
			
            //document.getElementById('collectBtn').addEventListener("click", wonder.getStories, true);
        },
		uid: { // Store Running User UID
		},
		uname: { // Store running User Username
		},
    },
    utils: {
        getExternalResource: function (url, cb) {
            GM_xmlhttpRequest({
                method: 'GET',
                url: url,
                onload: function (responseDetails) {
                    var dt = document.implementation.createDocumentType("html", "-//W3C//DTD HTML 4.01 Transitional//EN", "http://www.w3.org/TR/html4/loose.dtd"),
                        doc = document.implementation.createDocument('', '', dt),
                        html = doc.createElement('html');
                    html.innerHTML = responseDetails.responseText;
                    doc.appendChild(html);
                    cb(doc);
                }
            });
        },
        logger: function (msg, level) {
            var ts = new Date;
            var timestamp = ts.getFullYear() + "-" + wonder.utils.pad(ts.getMonth() + 1, 2) + "-" + wonder.utils.pad(ts.getDate(), 2) + " " + wonder.utils.pad(ts.getHours(), 2) + ":" + wonder.utils.pad(ts.getMinutes(), 2) + ":" + wonder.utils.pad(ts.getSeconds(), 2);
            var logLevels = {
                "debug": 0,
                "info": 1,
                "warn": 2,
                "error": 3
            };
            var minLogLevel = logLevels[logLevel];
            if (debug === false) {
                return true;
            }
            if (level === null || level === undefined) {
                level = "info";
            }
            else {
                level = level.toLowerCase();
            }
            if (level === "error" && minLogLevel <= 3) {
                console.error(timestamp + " - [ " + level.toUpperCase() + " ] - " + msg);
            }
            else if (level === "warn" && minLogLevel <= 2) {
                console.warn(timestamp + " - [ " + level.toUpperCase() + " ] - " + msg);
            }
            else if (level === "info" && minLogLevel <= 1) {
                console.info(timestamp + " - [ " + level.toUpperCase() + " ] - " + msg);
            }
			else if(level === "debug" && minLogLevel <= 0) {
				console.log(timestamp + " - [ " + level.toUpperCase() + " ] - " + msg);
			}
        },
        pad: function (number, length) {
            var str = '' + number;
            while (str.length < length) {
                str = '0' + str;
            }
            return str;
        },
        csv2Array: function (csv) {
            if (csv === '' || csv === null || csv === undefined) return new Array();
            var myArray = new Array();
            var t = csv.split(',');
            for (var i = 0; t.length > i; i++) {
                myArray.push(t[i]);
            }
            return myArray;
        },
        addGlobalStyle: function (css) {
            var head, style;
            head = document.getElementsByTagName('head')[0];
            if (!head) {
                return;
            }
            style = document.createElement('style');
            style.type = 'text/css';
            style.innerHTML = css;
            head.appendChild(style);
        },
        wipeDB: function () {
            var keys = GM_listValues();
            if (keys.length > 0) {
                wonder.utils.logger("Warning next run will collect items that have already been collected", "warn")
                for (var i = 0, key = null; key = keys[i]; i++) {
                    if (key.match(/runtime.completed/)) {
                        wonder.utils.logger("Removing Completed Data: " + key, "info");
                        wonder.opts.del(key);
                    }
                }
                return true;
            }
        },
        wait: function (check, callback) {
            if (check) {
				if(typeof callback == 'function') {
                  callback(); } 
            }
            else if(!check) {
                window.setTimeout('wonder.utils.wait.call(this.check, callback)', '400');
            }
        },
		pause: function (millis)	{		
			var date = new Date();
			var curDate = null;

			do { curDate = new Date(); }
			while(curDate-date < millis);	
		},
        fade: function (e, dir, s) {
            // Fade by JoeSimmons. Fade in/out by id and choose speed: slow, medium, or fast
            // Syntax: fade('idhere', 'out', 'medium');
            if (!e || !dir || typeof dir != 'string' || (dir != 'out' && dir != 'in')) {
                return;
            } // Quit if node/direction is omitted, direction isn't in/out, or if direction isn't a string
            dir = dir.toLowerCase();
            s = s.toLowerCase(); // Fix case sensitive bug
            var node = (typeof e == 'string') ? $(e) : e,
                // Define node to be faded
                speed = {
                    slow: 400,
                    medium: 200,
                    fast: 100
                };
            if (!s) var s = 'medium'; // Make speed medium if not specified
            if (s != 'slow' && s != 'medium' && s != 'fast') s = 'medium'; // Set speed to medium if specified speed not supported
            if (dir == 'in') node.style.opacity = '0';
            else if (dir == 'out') node.style.opacity = '1';
            node.style.display = '';
            var intv = setInterval(function () {
                if (dir == 'out') {
                    if (parseFloat(node.style.opacity) > 0) node.style.opacity = (parseFloat(node.style.opacity) - .1).toString();
                    else {
                        clearInterval(intv);
                        node.style.display = 'none';
                    }
                }
                else if (dir == 'in') {
                    if (parseFloat(node.style.opacity) < 1) node.style.opacity = (parseFloat(node.style.opacity) + .1).toString();
                    else {
                        clearInterval(intv);
                    }
                }
            }, speed[s]);
        },
		str2Currency: function (nStr) {
	  	  nStr += '';
		  x = nStr.split('.');
		  x1 = x[0];
		  x2 = x.length > 1 ? '.' + x[1] : '';
		  var rgx = /(\d+)(\d{3})/;
		  while (rgx.test(x1)) {
			x1 = x1.replace(rgx, '$1' + ',' + '$2');
		  }
		return x1 + x2;
		}
    },
    // End wonder.utils
    opts: {
        autoexpand: {},
        autolike: {},
        dimcompleted: {},
        config: function () {
            var cpanel = document.getElementById('cowwm-cpanel');
            var oPanel = document.getElementById('cowwm-config');
            var autoExpandEnabled = wonder.opts.get("opts.autoexpand");
            var autoLike = wonder.opts.get("opts.likeassist");
			var bonusLike = wonder.opts.get("opts.likebonus");
            var dimCompleted = wonder.opts.get("opts.dimcompleted");
			// Check and Set Auto Expand
            if (autoExpandEnabled === undefined) {
                var expandChecked = "checked";
                wonder.opts.autoexpand = true;
            }
            else if (autoExpandEnabled === true) {
                var expandChecked = "checked";
                wonder.opts.autoexpand = true;
            }
            else if (autoExpandEnabled === false) {
                var expandChecked = "";
                wonder.opts.autoexpand = false;
            }
			// Check and Set Like Marvels
            if (autoLike === undefined) {
                wonder.opts.autolike = true;
                var likeChecked = "checked";
            }
            else if (autoLike === true) {
                wonder.opts.autolike = true;
                var likeChecked = "checked";
            }
            else if (autoLike === false) {
                wonder.opts.autolike = false;
                var likeChecked = "";
            }
			// Check and Set Like Bonuses
			if (bonusLike === undefined) {
                wonder.opts.likebonus = true;
                var bLikeChecked = "checked";
            }
            else if (bonusLike === true) {
                wonder.opts.likebonus = true;
                var bLikeChecked = "checked";
            }
            else if (bonusLike === false) {
                wonder.opts.likebonus = false;
                var bLikeChecked = "";
            }
			// Check and Set Hide Posts
            if (dimCompleted === undefined) {
                var dimChecked = "checked";
                wonder.opts.dimcompleted = true;
            }
            else if (dimCompleted === true) {
                var dimChecked = "checked";
                wonder.opts.dimcompleted = true;
            }
            else if (dimCompleted === false) {
                var dimChecked = "";
                wonder.opts.dimcompleted = false;
            }

            if (oPanel === undefined || oPanel === null) {
                div = document.createElement('div');
                div.id = "cowwm-config";
                closeImg = "<img src='data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wCEAAkGBggGBQkIBwgKCQkKDRYODQwMDRoTFBAWHxwhIB8cHh4jJzIqIyUvJR4eKzssLzM1ODg4ISo9QTw2QTI3ODUBCQoKDAwFDQ4KDSkYHhgpKSkpKSkpKSkpKSkpKTUpKSkpKSkpKSkpKSkpKSkpKSkpKSkpKSkpKSkpKSkpKSkpKf/AABEIAA4ADgMBIgACEQEDEQH/xAAXAAADAQAAAAAAAAAAAAAAAAABAgcA/8QAIBAAAgMAAgEFAAAAAAAAAAAAAQIDBBEAIQUSQVFhgf/EABUBAQEAAAAAAAAAAAAAAAAAAAIB/8QAFBEBAAAAAAAAAAAAAAAAAAAAAP/aAAwDAQACEQMRAD8Ao965X8P42vZnqRvXxRK4A9SaOjmd9/fNcQSeJhlkqxwSOwJRSGzo9aBwS2KbGBbUDSvW6Ubq7mE5uH9HFUwzUkp0o2jSI6BIxOD43s+/IT//2Q==' id='cowwm-opts-close'/>";
                div.innerHTML = "<h4>CoW Wall Manager Options</h4>" + closeImg + "<br/><br/>" + "<label for='autoExpand' id='autoExpandLbl'>Auto Expand 'More Posts'</label>" + "<input type='checkbox' id='autoExpand' " + expandChecked + "/><br/>" + "<label for='bonusLike' id='bonusLikeLbl'>'Like' Bonuses Collected</label><input type='checkbox' id='bonusLike' " + bLikeChecked + "/><br/><label for='assistLike' id='assistLikeLbl'>'Like' Marvels Assisted</label><input type='checkbox' id='assistLike' " + likeChecked + "/><br/>" + "<label for='assistLike' id='dimCompletedLbl'>Hide Posts Already Processed</label><input type='checkbox' id='dimCollected' " + dimChecked + "/><br/>" + "<a href='javascript:void(0);' id='cowwm-resetdb'>Reset All Databases</a>" + indent + "<a href='javascript:void(0);' id='cowwm-reset-user-db'>Reset User Database</a>";
                cpanel.appendChild(div);
                $('cowwm-opts-close').addEventListener("click", wonder.opts.config, true);
                $('autoExpand').addEventListener('change', function () {
                    wonder.opts.set('opts.autoexpand', document.getElementById('autoExpand').checked);
                }, true);
                $('assistLike').addEventListener('change', function () {
                    wonder.opts.set('opts.likeassist', document.getElementById('assistLike').checked);
                }, true);
				 $('bonusLike').addEventListener('change', function () {
                    wonder.opts.set('opts.likebonus', document.getElementById('bonusLike').checked);
                }, true);
                $('dimCollected').addEventListener('change', function () {
                    wonder.opts.set('opts.dimcompleted', document.getElementById('dimCollected').checked);
                }, true);
                $("cowwm-resetdb").addEventListener("click", function () {
                    var r = confirm("Clearing the Database will attempt to Collect Bonuses and Assist with Marvels already Completed.\r\n\r\nDo you wish to Contiune?");
                    if (r == true) {
                        wonder.utils.wipeDB();
                    }
                }, true);
				 $("cowwm-reset-user-db").addEventListener("click", function () {
                    var r = confirm("Clearing the Database will attempt to Collect Bonuses and Assist with Marvels already Completed for " + wonder.core.uname + ".\r\n\r\nDo you wish to Contiune?");
                    if (r == true) {
                        wonder.opts.wipeUserData(wonder.core.uid);
                    }
                }, true);
            }
            else {
                if (oPanel.style.visibility == "hidden") {
                    oPanel.style.visibility = "visible";
                } else {
                    oPanel.style.visibility = "hidden";
                }
            }
        },
        get: function (key) {
            return GM_getValue(key);
        },
        set: function (key, value) {
			return GM_setValue(key, value);

        },
        del: function (key) {
            return GM_deleteValue(key);
        },
		wipeUserData: function (uid) {
			if(uid !== undefined) {
				console.log("Clearing Database for User: " + wonder.core.uname); 
				GM_deleteValue("runtime.completed.bonus." + uid);
				GM_deleteValue("runtime.completed.marvels." + uid); }
		},
    },
    // End wonder.opts
    actions: {
        // Start wonder.actions
        updatePanel: function (msg, status) {
            var cpanel = document.getElementById('cowwm-cpanel');
            if (msg === null || msg === undefined) {
                msg = "";
            }
            if (status === null || status === undefined) {
                status = "Idle";
            }
            if (typeof linkEnabled != 'boolean' || linkEnabled === "" || linkEnabled === undefined) {
                linkEnabled = true;
            }

            (status == "Idle" || status == "Paused") ? linkEnabled = true : linkEnabled = false;
			
            if (cpanel !== null && cpanel !== undefined) {
                var cowStatus = document.getElementById("cowwm-status");
                var btn = document.getElementById("collectBtn");

                if (cowStatus !== null && cowStatus !== undefined) {
                    cowStatus.innerHTML = status;
                }

                if (btn !== null && btn !== undefined) {
                    if (linkEnabled === true) {
                        btn.style.visibility = "visible";
                    }
                    else if (linkEnabled === false) {
                        btn.style.visibility = "hidden";
                    }
                    document.getElementById('collectBtn').addEventListener("click", wonder.actions.getStories, true);
                }
                document.getElementById('cowwm-logo').addEventListener("click", wonder.opts.config, true);
                document.getElementById('cowwm-panel-name').addEventListener("click", wonder.opts.config, true);
            }
        },
        click: function (e, type) {
            if (!e && typeof e == 'string') e = document.getElementById(e);
            if (!e) return;
            var evObj = e.ownerDocument.createEvent('MouseEvents');
            evObj.initMouseEvent("click", true, true, e.ownerDocument.defaultView, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
            e.dispatchEvent(evObj);
        },
        expand: function () {
            var el = document.body.getElementsByClassName('uiStreamCollapsed');
            for (var i = 0; el.length > i; i++) {
                wonder.utils.logger("Found More Posts Link - Expanding", "debug");
                wonder.actions.click(el[i].children[0]);
            }
        },
        getStories: function () {
            wonder.actions.updatePanel("", "Running");
            var myName = document.getElementsByClassName('fbxWelcomeBoxName')[0].text;
            var uid = document.getElementsByClassName('fbxWelcomeBoxName')[0].href.split("=")[1];
            wonder.utils.logger("Searching for new Posts", "info")
            // create some array's to store our data
            var lcount = new Array();
            var blinks = new Array();
            var mlinks = new Array();
            var tlinks = new Array();
            var myPosts = 0;
            var mDupeCount = 0;
            var bDupeCount = 0;
            // Get each Post for the Application
            var stories = document.getElementsByClassName('aid_' + applicationID);
            if (stories !== null && stories.length > 0) {
                for (var i = 0; i < stories.length; i++) {
                    var appid = stories[i].innerHTML.match(/(application\.php\?id=|app_id=)([0-9]{1,})/);
                    if (appid !== null && appid[2] === applicationID) {
                        var dataft = stories[i].attributes[1].nodeValue;
                        var fbid = JSON.parse(dataft)['fbid'];
                        var likeLink = stories[i].getElementsByClassName('like_link');
                        var links = stories[i].getElementsByTagName('a')
                        var actorName = links[2].text;
                        lcount.push(fbid)
                        if (actorName !== '') {
                            for (var x = 0; x < links.length; x++) {
                                var storyid = stories[i].id;
                                if (links[x].text.match(marvelRegex)) {
                                    k = uid + links[x].href.split('?')[0].split("/")[7] + links[x].href.split('?')[0].split("/")[8];
                                    if (wonder.utils.csv2Array(wonder.opts.get("runtime.completed.marvels." + uid)).indexOf(k) !== -1) {
                                        if (wonder.opts.dimcompleted === true) {
                                            wonder.utils.fade(storyid, "out", "fast");
                                        }
                                        mDupeCount++;
                                    }
                                    else {
                                        var mTmp = new Object;
                                        mTmp.actor = actorName
                                        mTmp.uid = uid;
                                        mTmp.type = links[x].href.split('marvelRequest-')[1].split('-')[0].replace('+', ' ');
                                        mTmp.id = fbid;
                                        mTmp.sid = storyid;
                                        mTmp.base = links[x].href.split('?')[0]
                                        mTmp.url = links[x].href
                                        mTmp.like_link = likeLink[0];
                                        wonder.utils.logger("Found Help Build " + mTmp.type + " Marvel Request for " + mTmp.actor + " Base: " + mTmp.base, "debug")
                                        mlinks.push(mTmp);
                                    }
                                } else if (links[x].text.match(bonusRegex)) {
                                    b = links[x].href.split('?')[0];
                                    k = uid + b.split("/")[7] + b.split("/")[8];
                                    if (wonder.utils.csv2Array(wonder.opts.get("runtime.completed.bonus." + uid)).indexOf(k) !== -1) {
                                        if (wonder.opts.dimcompleted === true) {
                                            wonder.utils.fade(storyid, "out", "fast");
                                        }
                                        bDupeCount++;
                                        continue;
                                    }
                                    var bTmp = new Object;
                                    bTmp.actor = actorName;
                                    bTmp.uid = uid;
                                    bTmp.type = links[x].text;
                                    bTmp.id = fbid;
                                    bTmp.sid = storyid;
                                    bTmp.base = links[x].href.split('?')[0]
                                    bTmp.url = links[x].href
                                    bTmp.like_link = likeLink[0];
                                    wonder.utils.logger("Found a Bonus (" + fbid + ") to Pickup from " + actorName, "debug")
                                    blinks.push(bTmp);
                                } else if (links[x].text.match(townRegex)) {
                                    wonder.utils.logger("Found Embassy Link", "debug");
                                    tlinks.push(links[x]);
                                    if (wonder.opts.dimcompleted === true) {
                                        wonder.utils.fade(storyid, "out", "fast");
                                    }
                                }
                            }
                        } else {
                            wonder.utils.logger("Skipping post from yourself", "debug");
                            myPosts++;
                            if (wonder.opts.dimcompleted === true) {
                                var storyid = stories[i].id;
                                wonder.utils.fade(storyid, "out", "fast");
                            }
                        }
                    }
                }
                // Done Collecting Data Start Process Links
                wonder.utils.logger("Found " + lcount.length + " Posts - Processing (" + mlinks.length + " Marvels, " + blinks.length + " Bonuses, and " + tlinks.length + " Embassy / Visit Links)", "info");
                // Process Marvels
                for (y = 0; mlinks.length > y; y++) {
                    var key = mlinks[y].uid + mlinks[y].base.split("/")[7] + mlinks[y].base.split("/")[8];
                    var mCompleted = wonder.utils.csv2Array(wonder.opts.get("runtime.completed.marvels." + uid));
					if(mCompleted.indexOf(key) === -1) {
                    var url = mlinks[y].url;
                    var id = mlinks[y].id;
                    var like = mlinks[y].like_link;
                    wonder.utils.logger("Attempting to Build " + mlinks[y].type + " for " + mlinks[y].actor, "info");
					
                    wonder.marvels.collect(id, url);
                    // Like item to mark to other user we helped out
                    if (wonder.opts.autolike === true) {
                        if (like.title == "Like this item") {
                            wonder.utils.logger("Marking Marvel (" + key + ") Assisted with 'Like'", "debug");
                            like.click();
                        }
                    }
                    if (wonder.opts.dimcompleted === true) {                       
                        wonder.utils.fade(mlinks[y].sid, "out", "fast");
                    }
                    wonder.utils.logger("Storing Marvel Data: " + key, "debug");
                    mCompleted.push(key);
                    wonder.opts.set("runtime.completed.marvels." + uid, mCompleted.toString());
					}
					else {
						wonder.utils.logger("Duplicate Marvel Request Found in Collection - Skipping","warn"); }
                }
                for (var z = 0; blinks.length > z; z++) {
                    var key = blinks[z].uid + blinks[z].base.split("/")[7] + blinks[z].base.split("/")[8];
                    var bonusURL = blinks[z].url;
                    var id = blinks[z].id;
					var like = blinks[z].like_link;
                    var bCompleted = wonder.utils.csv2Array(wonder.opts.get("runtime.completed.bonus." + uid));
                    wonder.utils.logger("Collecting Bonus from " + blinks[z].actor, "info");
                    wonder.bonus.collect(id, bonusURL);
					if (wonder.opts.likebonus === true) {
                        if (like.title == "Like this item") {
                            wonder.utils.logger("Marking Bonus (" + key + ") Assisted with 'Like'", "debug");
                            like.click();
                        }
                    }
                    if (wonder.opts.dimcompleted === true) {
                        wonder.utils.fade(blinks[z].sid, "out", "fast");
                    }
                    wonder.utils.logger("Storing Bonus Data " + key, "debug");
                    bCompleted.push(key);
                    wonder.opts.set("runtime.completed.bonus." + uid, bCompleted.toString());
					;
                }
                var mProcd = mlinks.length;
                var bProcd = blinks.length;
                var mDesc = "";
                var bDesc = "";
                if (mProcd > 0 && mDupeCount > 0) {
                    mDesc = "<b>Marvels</b> " + "Assisted: " + mProcd + " Skipped: " + mDupeCount + newLine;
                }
                else if (mProcd > 0 && mDupeCount == 0) {
                    mDesc = "<b>Marvels</b> " + "Assisted: " + mProcd + newLine
                }
                else if (mProcd <= 0 && mDupeCount > 0) {
                    mDesc = "<b>Marvels</b> " + "Skipped: " + mDupeCount + newLine;
                }
                if (bProcd > 0 && bDupeCount > 0) {
                    bDesc = "<b>Bonuses</b> " + "Collected: " + bProcd + " Skipped: " + bDupeCount + newLine;
                }
                else if (bProcd > 0 && bDupeCount == 0) {
                    bDesc = "<b>Bonuses</b> " + "Collected: " + bProcd + newLine
                }
                else if (bProcd <= 0 && bDupeCount > 0) {
                    bDesc = "<b>Bonuses</b> " + "Skipped: " + bDupeCount + newLine;
                }
                var desc = indent + "<b>Process Summary</b> (" + lcount.length + " Items)" + newLine + mDesc + bDesc + "<b>Visits</b> " + tlinks.length + " <b>Your Posts</b>: " + myPosts + "<br><br>";
                $("cowwm-summary").innerHTML = desc;
				wonder.reports.totals(uid);
                // Ugly Cleanup of Flash Loaded Iframes (Working on a better method)
				var tmout = (mlinks.length * 3000);

				if (tmout > 0 && mProcd > 0) {
					wonder.utils.logger("Running Clean up in " + tmout, "info"); 
				}
				wonder.actions.updatePanel(desc, 'Idle');
                //(mProcd > 0) ? setTimeout(wonder.actions.cleanup, tmout) : wonder.actions.updatePanel(desc, 'Idle')
            }
            else {
                wonder.utils.logger("No " + gameTitle + " Posts Found", "warn")
                wonder.actions.updatePanel('');
            }
        },
        cleanup: function (el,intId) {
            wonder.utils.logger("Running Cleanup", "info");
			try	{
				var cp = document.getElementById("cowwm-cpanel")
				var iframes = cp.getElementsByTagName("iframe")
	            while (iframes.length > 0) {
		            for (var i = 0; iframes.length > i; i++) {
			            wonder.utils.logger("Removing Stale Marvel Assist Frame " + iframes[i].id, "info");
	                    $("cowwm-cpanel").removeChild(iframes[i]);
	                }
		        }
			    var totals = $("cowwm-summary").innerHTML
	            wonder.actions.updatePanel(totals, 'Idle');
				
				}
			catch (e)
			{
				console.log("%s", e);
			}
        }
    },
    // End wonder.actions
    // Start wonder.marvels
    marvels: {
        collect: function (id, url) {
            getDOC(url, function (resp, url) {
                var nextURL = resp.getElementsByTagName('iframe')[0].src;
                var oURL = url.match(/(.*)\?(.*)/);
                var tFrame = document.getElementsByTagName('iframe')[0].src;
                var tURL = nextURL.match(/(.*)\?/)
                var query = nextURL.match(/(.*)\&(fb_sig_in_iframe=.*)/);
                var finalURL = tURL[1] + "/?" + oURL[2] + "&" + query[2];
                //console.log("Final Marvel URL: " + finalURL);
                //GM_xmlhttpRequest({
                //    method: "GET",
                //    url: finalURL,
                //    onload: function (details) {
                        var iframe = document.createElement("IFRAME");
                        $("cowwm-cpanel").appendChild(iframe);
                        iframe.contentWindow.location.href = 'about:blank';
                        //iframe.contentDocument.open("text/html");
                        //iframe.contentDocument.write(details.ResponseText);
                        //iframe.contentDocument.close();
                        iframe.src = finalURL;
                        iframe.style.position = "absolute";
                        iframe.style.width = "1px";
                        iframe.style.height = "1px";
                        iframe.id = id;
                 //   }
               // });
            });
        },
    },
    bonus: {
        collect: function (key, url) {
            wonder.utils.getExternalResource(url, function (doc) {
                var tFrame = doc.getElementsByTagName('iframe');
                var query = tFrame[0].src.match(/(.*)\&(fb_sig_in_iframe=.*)/);
                var appendID = tFrame[0].src.match(/(bapiTicketId=)(.*)(\&ref=)/);
                var uri = tFrame[0].src.match(/(.*)\?/)
                if (appendID != null && uri != null) {
                    var newURL = uri[1] + "/" + appendID[2]
                    //console.log("Append: " + appendID[2] + " URI: " + uri[1]); 
                    newURL = newURL + "/?" + query[2];
                    newURL = newURL.replace("/bonusableFeed/", "/bonusableFeedHelper/")
                    //console.log("Bonus Helper URL: " + newURL)
                    wonder.utils.getExternalResource(newURL, function (result) {
                        //console.log(result);
                        wonder.utils.logger("Done Collecting Bonus " + key, "debug");
                    });
                }
            });
        },
    },
    worker: {
        create: {},
        status: function (polerID) {
			if(wonder.opts.get("worker.status") == "success") {
				wonder.utils.cleaup();
				cancelInterval(pollerID);
			}
		},
        add: {},
        del: {},
    },
    // End wonder.marvels
	reports: { 
		totals: function (uid) {
			if(uid === undefined || uid === null || uid === '') { return false; }

			var mCompleted = wonder.utils.csv2Array(wonder.opts.get('runtime.completed.marvels.' + uid));
			var bCompleted = wonder.utils.csv2Array(wonder.opts.get('runtime.completed.bonus.' + uid));
			var silver = (mCompleted.length * 200) + (bCompleted.length * 500);

			$("cowwm-totals-marvels").innerHTML = mCompleted.length;
			$("cowwm-totals-bonus").innerHTML = bCompleted.length;
			$("cowwm-totals-silver").innerHTML = wonder.utils.str2Currency("~ $" + silver);
		}
	}
} // End Wonder
//wonder.cleanup();
if (wonder.core.init()) {
	    wonder.utils.logger("City of Wonder Init Successful", "info");
    //wonder.expand();

    window.addEventListener("load", function () {
	
	if(wonder.opts.get("opts.autoexpand") === true) {
		setInterval(wonder.actions.expand, 1000); }
	
	//console.clear();
	wonder.utils.addGlobalStyle(

		    '#collectBtn {' + 
				'color: orange; }' + 
					
			'#cowwm-panel-name, #cowwm-logo {' +
				'  cursor: pointer; }' +

			'#cowwm-resetdb {' +
				'  float: left;' +
				'  padding-left: 10px; }' +

			'#cowwm-reset-user-db {' +
				'  float: right;' +
				'  padding-right: 10px; }' +

			'#cowwm-resetdb, #cowwm-reset-user-db {' +
				'  position: relative;' +
				'  bottom: -30px;' +
				'  color: white; }' +

			'#cowwm-opts-close {' +
				'  position: absolute;' +
				'  top: 5px;' +
				'  right: 5px; }' +

			'#cowwm-cpanel {' + 
				' -moz-border-radius-topright: 5px;' +
				' -moz-border-radius-topleft: 5px;' +
				'  position: fixed;' + 
				'  left: 20px;' + 
				'  right: 0;' + 
				'  bottom: 0;' + 
				'  top: auto;' + 
		//		'  border-top: 1px solid silver;' + 
				'  background: black;' + 
				'  color: white;' + 
				'  margin: 1em 0 0 0;' + 
		//		'  padding: 2px 0 0.4em 0;' + 
				'  width: 220px;' +
				'  text-align: center; ' +
				'  z-index: 99999;' +
				'  font-family: Verdana, sans-serif; }' +

			'#cowwm-logo {'+
				'  position: absolute;' +
				'  bottom: 2px;' +
				'  left: 3px;' +
				'  border: 0 none; }' +

			'#cowwm-cpanel iframe {' +
				'  position: absolute;' +
				'  width: 1px;' +
				'  height: 1px' +
				'  visibility: hidden;' +
				'  z-index: -99999; }' +

			'#cowwm-config {' +
				' -moz-border-radius-topright: 5px;' +
				' -moz-border-radius-topleft: 5px;' +
				' -moz-border-radius-bottomright: 5px;' +
				' -moz-border-radius-bottomleft: 5px;' +
				'  width: 300px;' +
				'  height: 150px;' +
				'  position: fixed;' +
				'  text-align: left;' +
				'  top: 50%; left: 40%;' +
				'  background-color: black; }' +

			'#cowwm-config h4 {' +
				' text-align: center;' +
				' -moz-border-radius-topright: 3px;' +
				' -moz-border-radius-topleft: 3px;' +
				'  text-color: white; }' +

			'#cowwm-config label {' +
				'  color: white;' +
				'  margin-top: 2px;' +
				'  font-family: Verdana, sans-serif;' +
				'  font-weight: normal;' +
				'  display: normal;' +
				'  position: absolute;' +
				'  left: 10px; }' +

			'#cowwm-config input {' +
				' width: 13px;' +
				' height: 13px;' +
				' padding: 0;' +
				' vertical-align: middle;' +
				' position: absolute;' +
				' right: 10px; }' +

			'#cowwm-cpanel h4, cowwm-totals h4 {' +
				' -moz-border-radius-topright: 5px;' +
				' -moz-border-radius-topleft: 5px;' +
				'  background-color: #ff9900;' +
				'  height: 20px;' +
				'  padding-top: 5px;' +
				'  text-align: center;' +
				'  text-color: white; ' +
				'  color: white; }' +

			'#cowwm-summary {' +
				'  background-color: black;' +
				'  width: 220px;' +
				'  position: relative;' +
				'  text-align: left;' +
				'  top: bottom: 50px; }' +

			'#cowwm-totals {' +
				'  text-align: left; }' +

			'#cowwm-totals label {' +
				'  color: white;' +
				'  font-weight: bold;' +
				'  padding-left: 10px;' +
				'  text-align: left; }' +

			'#cowwm-totals span {' +
				'  font-weight: normal; }'
				
				//'  font-size: small;' + 
				//'  line-height: 160%;' + '}' + 
			);
			wonder.actions.updatePanel("","Idle");
			
test("http://socialciv-fb-web-active-vip.playdom.com/socialciv/fb/newsfeed/helpMarvelBuild/4619015/337432/84208021?ref_id=563081194&send_timestamp=1284020352&track=newsfeed-marvelRequest-Fort%20Knox-0&ref=nf&fb_sig_in_iframe=1&fb_sig_base_domain=socialciv-fb-web-active-vip.playdom.com&fb_sig_locale=en_US&fb_sig_in_new_facebook=1&fb_sig_time=1284020611.8378&fb_sig_added=1&fb_sig_profile_update_time=1282115230&fb_sig_expires=1284026400&fb_sig_user=100000420362870&fb_sig_session_key=2.pCvgb3E9tgTvyfptJfe4Yw__.3600.1284026400-100000420362870&fb_sig_ss=ENXlQu_pv8JQTGlb_S2q0A__&fb_sig_cookie_sig=bf342281dc6536295cc0557917ed042c&fb_sig_country=us&fb_sig_api_key=665a04a75b148852fda41a0b3b491c77&fb_sig_app_id=114335335255741&fb_sig=7eea1ae085901c3d2f66d35c166e8bf4&_=1284020828770", doc_ready());


	}, true);

}

function test (url) {
				var uid = document.getElementsByClassName('fbxWelcomeBoxName')[0].href.split("=")[1];
				var id = "1234569";
                var t = 'javascript:(' + function () { console.log("Frame Loaded"); document.domain = "http://socialciv-fb-web-active-vip.playdom.com" } + ')()';
                var iframe = document.createElement("IFRAME");					
                        iframe.style.position = "absolute";
                        iframe.style.width = "1px";
                        iframe.style.height = "1px";
                        iframe.id = id;
						iframe.name = 'collector-' + id;
						iframe.src = url;
						$("cowwm-cpanel").appendChild(iframe);
}

function doc_ready () { 
	if ((""+window.location).toLowerCase().indexOf("http://socialciv-fb-web-active-vip.playdom.com/socialciv/fb/newsfeed/helpmarvelbuild/") != -1) {
		console.log("Running Check: " + self.name);
		var msg = document.getElementById("mainBG").innerHTML
		if(msg !== undefined) {
		  if(msg.search("to see this incredible wonder") !== -1)  {
			console.log("Failed - Completed already");
			wonder.opts.set("worker.status." + self.name, "failure"); 
		  } else if (msg.search("you have already helped") !== -1) {
			  console.log("Failed already helped")
			wonder.opts.set("worker.status." + self.name, "failure"); 
		  } else if (msg.search("Collect Reward") !== -1) {
			  console.log("Success");
			wonder.opts.set("worker.status." + self.name, "success");
		  } else if (msg.search("You helped build") !== -1) {
			console.log("Success");
			wonder.opts.set("worker.status." + self.name, "success");
		  } else { console.log("IDK"); }
	}  }
	else if (document.getElementsByClassName("fbxWelcomeBoxName").length > 0) {
		var uid = document.getElementsByClassName('fbxWelcomeBoxName')[0].href.split("=")[1];
		var ifs = $("cowwm-cpanel").getElementsByTagName("iframe");
		for(var i = 0; ifs.length > i; i++) {
			var worker = "worker.status." + ifs[i].name;
			var result = wonder.opts.get(worker)
			console.log("Checking worker status: " + worker + " " + result);
			if(wonder.opts.get("worker.status." + ifs[i].name) !== undefined) {
				console.log("Collector Result: " + result);
				wonder.opts.del(worker);
				console.log("Removing Stale Frame");
			$("cowwm-cpanel").removeChild(ifs[i]); }
		}
	}
}	
setInterval(function () { doc_ready() }, 1000)