
var BROWSER = {
	PC : {
		isIE : function() {
			return ( !BROWSER.isMobile() && navigator.appName == 'Microsoft Internet Explorer');
		},
		
		isFirefox : function() {
			return ( !BROWSER.isMobile() && navigator.userAgent.indexOf('Firefox') > -1 );
		},
		
		isChrome : function() {
			return ( !BROWSER.isMobile() && navigator.userAgent.indexOf('Chrome') > -1 );
		},
		
		isSafari : function() {
			return ( !BROWSER.isMobile() && navigator.userAgent.indexOf('Safari') > -1 );
		},
		
		isOpera : function() {
			return ( !BROWSER.isMobile() && navigator.userAgent.indexOf('Opera') > -1 );
		}
	},
	
	MOBILE : {
		isAndroid : function() {
			return ( navigator.userAgent.indexOf('Android') > -1 );
		},
		
		isIOS : function() {
			return ( navigator.userAgent.indexOf('iPad') > -1 ) || ( navigator.userAgent.indexOf('iPhone') > -1 ) || ( navigator.userAgent.indexOf('iPod') > -1 );
		},
		
		isWindowMobile : function() {
			return ( navigator.userAgent.indexOf('IEMobile') > -1 );
		},
		
		isGalaxyNote3 : function () {
			return ( navigator.userAgent.indexOf('SM-N900') > -1 );
		}
	},
	
	isMobile : function() {
		return BROWSER.MOBILE.isAndroid() || BROWSER.MOBILE.isIOS() || BROWSER.MOBILE.isWindowMobile();
	},
	
	VERSION : {
		firefox : function() {
			var ua = navigator.userAgent;
			var versionIdx = ua.indexOf('Firefox/') + 8;
			var versionStr = ua.substr(versionIdx);
			versionStr = versionStr.substr(0, versionStr.indexOf('.'));
			try {
				return parseInt(versionStr);	
			} catch (e) {
				// nothing
			}
			return 0;
		},
		IE : function() {
			var ua = navigator.userAgent;
			// find Trident Engine Version
			var engineIdx = ua.indexOf('Trident/') + 8;
			var engineVerStr = ua.substr(engineIdx);
			engineVerStr = engineVerStr.substr(0, engineVerStr.indexOf('.'));
			try {
				var engineNum = parseInt(engineVerStr);	
				if(engineNum==7) { return 11;
				} else if(engineNum==6) { return 10; 
				} else if(engineNum==5) { return 9; 
				} else if(engineNum==4) { return 8; 
				} else if(engineNum==3) { return 7; }
			} catch (e) {
				// maybe IE-6
			}
			
			// find MSIE compatible
			var versionIdx = ua.indexOf('MSIE ') + 5;
			var versionStr = ua.substr(versionIdx);
			versionStr = versionStr.substr(0, versionStr.indexOf('.'));
			try {
				return parseInt(versionStr);	
			} catch (e) {
				// nothing
			}
			return 0;
		},
		android : function() {
			var ua = navigator.userAgent;
			var versionIdx = ua.indexOf('Android ') + 8;
			var versionStr = ua.substr(versionIdx, 1);
			try {
				return parseInt(versionStr);	
			} catch (e) {
				// nothing
			}
			return 0;
		},
		IOS : function() {
			var ua = navigator.userAgent;
			//var versionIdx = ua.indexOf('CPU OS ') + 7;
			var versionIdx = ua.indexOf('OS ') + 3;
			var versionStr = ua.substr(versionIdx, 1);
			try {
				return parseInt(versionStr);	
			} catch (e) {
				// nothing
			}
			return 0;
		},
		isIPhone6Plus : function () {
			var ua = navigator.userAgent;
			if (ua.indexOf('iPhone') > 1) {
				if (screen.availWidth == 736
					&& screen.availHeight == 414) {
					return 1;	
				}
			}
			return 0;
		},
		isAndroidICS : function() {
			if (BROWSER.MOBILE.isAndroid()){
				var ua = navigator.userAgent;
				var versionIdx = ua.indexOf('Android ') + 8;
				var versionStr = ua.substr(versionIdx, 3);
				if (versionStr =='4.0'){
					return true;
				}
			}
			return false;	
		},
		isAndroidJellyBean : function() {
			if (BROWSER.MOBILE.isAndroid()){
				var ua = navigator.userAgent;
				var versionIdx = ua.indexOf('Android ') + 8;
				var versionStr = ua.substr(versionIdx, 3);
				if (versionStr =='4.1' || versionStr =='4.2' || versionStr =='4.3'){
					return true;
				}
			}
			return false;	
		}
	},
	
	GRAPHIC : {
		isDrawCanvas : function() {
			return (BROWSER.PC.isFirefox() && BROWSER.VERSION.firefox() < 4) // FF v3.0 less
				|| (BROWSER.MOBILE.isAndroid() && BROWSER.VERSION.android() < 3) // android v2.0 less
				|| (BROWSER.MOBILE.isIOS() && BROWSER.VERSION.IOS() < 5); // IOS v4.0 less
		},
		
		isDrawSVG : function() {
			return BROWSER.PC.isSafari() // safari
				|| BROWSER.PC.isChrome() // chrome
				|| BROWSER.PC.isOpera() // opera
				|| (BROWSER.PC.isFirefox() && 4 <= BROWSER.VERSION.firefox()) // FF v4.0 higher
				|| (BROWSER.MOBILE.isIOS() && 5 <= BROWSER.VERSION.IOS()) // IOS v5.0 higher
				|| BROWSER.MOBILE.isWindowMobile() // window mobile
				|| (BROWSER.MOBILE.isAndroid() && 3 <= BROWSER.VERSION.android()); // android v3.0 higher
		},

		isHeaderAbsoluteUse : function() {
			var ua = navigator.userAgent;
			if (BROWSER.MOBILE.isAndroid())
			{
				var versionIdx = ua.indexOf('Android ') + 8;
				var versionStr = ua.substr(versionIdx, 3);
				try {
					var ver = parseFloat(versionStr);
					if(ver == '4.0' || ver >= '4.2'){
						return 1;
					}
				} catch (e) {
					// nothing
				}
			}
			else if (BROWSER.MOBILE.isIOS())
			{
				if(BROWSER.VERSION.isIPhone6Plus()) {
					return 1;
				}
			}
			return 0;
		},
		
		isHeaderFixedUse : function() {
			if (BROWSER.MOBILE.isAndroid())
			{
				var ua = navigator.userAgent;
				var versionIdx = ua.indexOf('Android ') + 8;
				var versionStr = ua.substr(versionIdx, 3);
				try {
					var ver = parseFloat(versionStr);
					if(ver == '4.1'){
						return 1;
					}
				} catch (e) {
					// nothing
				}
			}
			return 0;
		},
		isDrawVML : function() {
			return BROWSER.PC.isIE();
		},
		isNotIFrame : function() {
			var ua = navigator.userAgent;
			if (BROWSER.MOBILE.isIOS())
			{
				var versionIdx = ua.indexOf('OS ') + 3;
				var versionStr = ua.substr(versionIdx, 1);
				try {
					var ver = parseInt(versionStr);
					if(ver < '7'){
						return 1;
					}else{
						var versionStr2 = ua.substr(versionIdx, 3);
						if (versionStr2 =='7_0' || versionStr2 =='7_1'){
							return 1;
						}
					}
				} catch (e) {
				}
			}
			return 0;
		}
	},

	adjustXhtml : function() {
		return !(BROWSER.PC.isIE() && BROWSER.VERSION.IE()<10);
	},

	adjustHtml : function() {
		return !BROWSER.adjustXhtml();
	}
}
