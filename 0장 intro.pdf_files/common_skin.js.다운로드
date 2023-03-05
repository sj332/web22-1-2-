// TODO: localSynap func 
// TODO: mobile용과 desktop용 동작을 염두해야함

// Header
function isHeader(){
	if ($('#header').length==0){
		return false;
	}else{
		return true;
	}
}
function getHeaderWidth(){
	if (isHeader()){
		return $('#header').width();
	}else{
		return 0;
	}
}
function setHeaderWidth(w){
	if (isHeader()){
		$('#header').width(w);
	}
}
function getHeaderHeight(){
	if (isHeader()){
		return $('#header').height();
	}else{
		return 0;
	}
}
// HeaderWrap
function getHeaderWrapHeight(){
	return $('#headerWrap').height();
}
// Footer
function isFooter(){
	if ($('#footerWrap').length==0){
		return false;
	}else{
		return true;
	}
}	
function getFooterHeight(){
	if (isFooter()){
		return $('#footerWrap').height();
	}else{
		return 0;
	}
}

$.fn.toggleCss = function(css, val1) {
	cur_val = this.css(css);
	if (parseInt(cur_val, 10) == 0) {
		this.css(css, val1);
	} else {
		this.css(css, 0);
	}
	return this;
}

/////////////////////////////////////////////////////////// 데스크탑 Start

/////////////////////////////////////////////////////////// 데스크탑 End

/////////////////////////////////////////////////////////// 모바일 start
function initMobile(){
	if( BROWSER.MOBILE.isIOS() ){
		$('.btn_close').show();
		$('.btn_close').on('click', function(e){
			e.preventDefault();
			window.close();
		});
	}
	if (BROWSER.MOBILE.isAndroid()
		|| BROWSER.GRAPHIC.isHeaderAbsoluteUse()) {
		// 2015.05.12 header 조절 없도록. 안드로이드는 absolute, iOS는 fixed로
		//if(BROWSER.VERSION.isAndroidICS()) {
			$('#headerWrap').css('position', 'absolute');
			$('#header').css('position', 'absolute');
			$('#header').css('width', '100%');
		//}
	}
	// 노트3는 버튼 absolute해야함
	if (BROWSER.MOBILE.isGalaxyNote3() || BROWSER.VERSION.IOS() == 7) {
		$('.btn_pre').css('position', 'absolute');
		$('.btn_next').css('position', 'absolute');
	}
	if ($('.select1').length > 0 && (localSynap.getFileType() === "xls" || localSynap.getFileType() === "xlsx")){
		// #DEFECT-2501 select1도 absolute로 해서 크기가 클때 밀리지 않게 해야함.	
		if (BROWSER.MOBILE.isGalaxyNote3() || BROWSER.VERSION.IOS() == 7) {
			$('.select1').css('position', 'absolute');
		}
		// 글씨가 길어지는 것은 셀에서만 발생
		// TODO : headLogo 태그가 존재하지 않을때, headRight 태그가 존재하지 않을때 예외처리
		// FIXME : 60 은 의미있는 숫자로 부여합니다.
		//var w = $(window).width() - $('#headLogo')[0].clientWidth - $('#headRight')[0].clientWidth - 60;
		var w = $(window).width() - $('#headLogo').width() - $('#headRight').width() - 60;
		$('.select1').css('max-width',w);
	}
}

function initSingle() {
	$('#headerWrap').css('position', 'absolute').css('z-index', 10000000000);
	$('#header').css('position', 'absolute');
	if ($('#fullScreenToolBar').length>0){
		$('#fullScreenToolBar').css('position','absolute');
	}
}

function setResizeHeaderTitle(){
	if ($('#headTitle').length > 0){
		var window_w = $(window).width() - $('#headLogo').width() - $('.navigation').width() - 60;
		// FIXME : -60은 의미있는 숫자로 부여합니다.
		$('#headTitle').width(window_w);
	}
}

function setDownloadButton(obj) {
	var url;
	if ($('#downloadBtn').length!=0){
		if(typeof obj==="string") {
			url = obj;
		} else {
			url = $(obj).find('downUrl') && $(obj).find('downUrl').text();
		}
		var $href = $('#downloadBtn');
		if( !url ){
			$href.css('display', 'none');
		}
		else{
			$href.attr('href', url).css('display', 'initial');
		}
	}
}

function printWordIframe(){
	if (BROWSER.PC.isChrome()) {
		$('#innerWrap').focus();
		frames[0].print();
	}else{
		var frm = document.getElementById("innerWrap").contentWindow;
		frm.focus();
		frm.print();
	}
}

function setPrintButton(arg) {
	if (localSynap.properties.isRenderServer){
		$('#printBtn').attr('href', 
						'javascript:api.printDocFromServer(\'' + localSynap.jobId + '\',\'' + localSynap.properties.contextPath + '\')'
						);
	}else{
		// 브라우저 인쇄 사용
		if(localSynap.properties.allowPrint && (localSynap.getFileType() === "doc" || localSynap.getFileType() === "docx" || localSynap.getFileType() === "hwp2k" || localSynap.getFileType() === "hwp97" || localSynap.getFileType() === "txt")){
			$('#printBtn').attr('href', 'javascript:printWordIframe()');
		}
		else{
			$('#printBtn').css('display', 'none');
		}
	}
}

function afterSingle() {
	var doc_w = $(document).width();
	if (localSynap.isSingleLayout()){
		if (doc_w > $('#headerWrap').width()){
			$('#headerWrap').width(doc_w);
		}
	}else{
		if ( BROWSER.GRAPHIC.isDrawCanvas() && (doc_w>$('#headerWrap').width())) { // Android 2.3.6
			$('#headerWrap').width(doc_w);
		}
	}
}

/////////////////////////////////////////////////////////// 모바일 End

/** FullScreen 관련 시작 **/

if (BROWSER.isMobile()) {

}else {
localSynap = (function() {
	var private = {
		toggleFullScreen: function () {
			$('.fullScreenOnly').toggle(!localSynap.fullScreenMode);
			$('.normalScreenOnly').toggle(localSynap.fullScreenMode);
			localSynap.fullScreenMode = !localSynap.fullScreenMode;
			return localSynap.fullScreenMode;
		},

		toggleFS: function() {
			if (localSynap.getFileType() !== "pdf"){
				if (localSynap.isSlide()){
					$('#container').toggleCss('top', localSynap.topS+'px').toggleCss('bottom', localSynap.bottomS+'px');
				}else{
					$('#container').toggleCss('padding-top', localSynap.topS+'px').toggleCss('padding-bottom', localSynap.bottomS+'px'); 
				}
			}
			private.toggleFullScreen();
			if (localSynap.isSlide() || localSynap.getFileType() === "image"){
				$(window).resize();
			} else if (localSynap.getFileType() === "pdf"){
				resizeView();
			}
			return false;
		}
	};

	var public = {
		fullScreenMode: false,
		prevThumbOpen: true,
		goFullScreen: (function() {
			if (BROWSER.isMobile()) {
				return function() {
				};
			}
			else {
				return function() {
					if (!BROWSER.PC.isIE()) {
						// TODO: get Document Element API to common.js or format.js
						if (parseInt(($("#thumbnail").css('left'))) < 0) {
							localSynap.prevThumbOpen = false;
						} else {
							localSynap.prevThumbOpen = true;
						}
						$('.closeBtn').click();
					} else {
						localSynap.fullScreenMode = true;
						$('.normalScreenOnly').hide();
						$('.fullScreenOnly').show();
					}
					$('#documentScale').removeClass('normalScaleTop').addClass('fullscreenScaleTop');
					$('#container').css('top', 0).css('bottom', 0);
					$(window).resize();
					return false;
				};
			}
		})(),
		exitFullScreen: (function() {
			if (BROWSER.isMobile()) {
				return function() {
				};
			}
			else {
				return function () {
					if (!BROWSER.PC.isIE()) {
						if (localSynap.prevThumbOpen === true) {
							$('.openThumbnail').click();
						} else {
							$('.closeBtn').click();
						}
					} else {
						localSynap.fullScreenMode = false;
						$('.fullScreenOnly').hide();
						$('.normalScreenOnly').show();
					}
					containerSizeAdjust();
					$('#documentScale').removeClass('fullscreenScaleTop').addClass('normalScaleTop');
					localSynap.thumbnailScrollPositionFix && localSynap.thumbnailScrollPositionFix();
					$(window).resize();
					return false;
				};
			}
		})()
	};

	return $.extend(localSynap, public);
})();

	$(document).ready(function() {
		if (!BROWSER.PC.isIE()) {

			document.addEventListener("fullscreenchange", function () {
				localSynap.fullScreenMode = document.fullscreen;
			    if (localSynap.fullScreenMode) {
                    localSynap.goFullScreen();
                } else {
                    localSynap.exitFullScreen();
                }
			}, false);
			document.addEventListener("mozfullscreenchange", function () {
				localSynap.fullScreenMode = document.mozFullScreen;
				if (localSynap.fullScreenMode) {
                    localSynap.goFullScreen();
                } else {
                    localSynap.exitFullScreen();
                }
			}, false);
			document.addEventListener("webkitfullscreenchange", function () {
				localSynap.fullScreenMode = document.webkitIsFullScreen;
				if (localSynap.fullScreenMode) {
                    localSynap.goFullScreen();
                } else {
                    localSynap.exitFullScreen();
                }
			}, false);
		}
	});
}
/** 데스크탑용 FullScreen 관련 끝 **/
function setFullScreenClick(){
	if( BROWSER.isMobile() ){
	}else{
		var showFullScreen = function () {
				$('#documentScale').removeClass('normalScaleTop').addClass('fullscreenScaleTop');
				$('#container').css('top', 0).css('bottom', 0);
		};
		var closeFullScreen = function () {
				containerSizeAdjust();
				$('#documentScale').removeClass('fullscreenScaleTop').addClass('normalScaleTop');
				localSynap.thumbnailScrollPositionFix && localSynap.thumbnailScrollPositionFix();
		};
		if (!BROWSER.PC.isIE()) {
			document.addEventListener("fullscreenchange", function () {
				if (document.fullscreenElement != null) {
					showFullScreen();
				} else {
					closeFullScreen();
				}
				$(window).resize();
			});
		}
		$('#fullScreenBtn').on('click', function () {
            var docElm = top.document.documentElement;
            if (docElm.requestFullscreen) {
                docElm.requestFullscreen();
            } else if (docElm.mozRequestFullScreen) {
                docElm.mozRequestFullScreen();
            } else if (docElm.webkitRequestFullScreen) {
                docElm.webkitRequestFullScreen();
            } else if (docElm.msRequestFullscreen) {
                docElm.msRequestFullscreen(); 
			} else {
				localSynap.fullScreenMode = true;
				$('.normalScreenOnly').hide();
				$('.fullScreenOnly').show();
				showFullScreen();
				$(window).resize();
			}
        });
		$(document).delegate('#exitFullScreenBtn', 'click', function () {
			var docElm = document;
			if (docElm.cancelFullScreen) {
				docElm.cancelFullScreen();
			} else if (docElm.mozCancelFullScreen) {
				docElm.mozCancelFullScreen();
			} else if (docElm.webkitCancelFullScreen) {
				docElm.webkitCancelFullScreen();
			} else if (docElm.msExitFullscreen) {
				docElm.msExitFullscreen();
			} else {
				localSynap.fullScreenMode = false;
				$('.fullScreenOnly').hide();
				$('.normalScreenOnly').show();
				closeFullScreen();
				$(window).resize();
			}
		}); 
		
		
		$('.exitscreen img').on('mouseover', function(){
			$(this).removeClass('mouseOut').addClass('mouseOn');
		}).on('mouseout', function(){
			$(this).removeClass('mouseOn').addClass('mouseOut');
		}).addClass('mouseOut');
	}
}
/** FullScreen 관련 끝 **/

// EVENT STOPPER
/** 복사방지호출 스크립트 시작 **/
// TODO: MOVE LOCAL SYNAP FUNC AT FILETYPE.js
function setKillCopyHtml(fileType){
	if (fileType=="image"){
		localSynap = getSynapPageObject();
		localSynap.killCopyHtml = function() {
			// common
			stopBrowserEvent(document.body);
			//innerWrap
			var contentBody = document.getElementById('content_body');
			if(contentBody) {
				contentBody.onload = function() {
					stopBrowserEvent(contentBody);
					if(BROWSER.isMobile()) {
						var src = contentBody.getAttribute('src');
						contentBody.removeAttribute('alt', '');
						contentBody.removeAttribute('src', '');
						contentBody.setAttribute('style', contentBody.getAttribute('style') + 'background-image:url(' + src + ');');
					}
				}
			}
		};
	} else if (fileType=="doc" || fileType=="docx" || fileType=="hwp2k" || fileType=="hwp97" || fileType=="ndoc" || fileType=="txt"){
	}else if (fileType=="ppt" || fileType=="pptx"){
	}else if (fileType=="xls" || fileType=="xlsx"){
	}else if (fileType=="pdf"){
		
	}
}
/** 복사방지호출 스크립트 끝 **/

function containerSizeAdjust(){
	$('#container').css('top', getHeaderHeight() + 'px').css('bottom', getFooterHeight() + 'px');
	if( $('#container').height() === 0 ){		// bottom : 0 style이 적용되지 않는 경우 (ex Android kitkat webview)
		$('#container').height( window.innerHeight - getHeaderHeight() );
		fn = window.onresize;
		window.onresize = function(){
			fn();
			$('#container').height( window.innerHeight - getHeaderHeight() );
		}
	}
}

function getInnerHeight() {
	var h;
	if (BROWSER.MOBILE.isAndroid()){
		h = screen.height;
	}else{
		h = window.innerHeight;
	}	
	return h;
}

/*window.onbeforeunload = function(e){
	if (typeof localSynap.jobId != "undefined" && typeof localSyanp.properties.contextPath != "undefined" ) {
		var url = localSyanp.properties.contextPath +'/status/close';
		$.ajax({
			type: 'GET',
			async: true,
			url: url,
			dataType: 'xml',
			success: function(xmlDoc){
			},
			error: function(error){
			}
		});
	}
}*/
