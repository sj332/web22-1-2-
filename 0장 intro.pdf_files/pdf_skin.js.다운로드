// pdf_skin.js
console.log( 'pdf_skin.js' );

var defaultDPI = "M";
var enableScrollEvent = true;
var enableZoomEvent = true;
var LIMIT_NUMBER = 999999;

$.extend(localSynap, (function() {
	var member = {
		// jQuery객체에 접근하기 위한 변수(성능관련)
		$idxFrame: undefined,
		$pdfFrame: undefined,
		// 화면에 표시되는 영역을 관리하는 변수
		arrThumbShowList: [],
		arrPageShowList: [],

		// 방문했던 페이지를 관리하기 위한 변수.. 같은데 아마도 이젠 필요 없을 것 같다.
		arrVisitPageIdx: [],

		// 페이지의 너비/폭, 인덱스, 경로 등 XML데이터를 담을 리스트
		rawObjList: [],

		// 각 페이지 bottom의 스크롤 높이. resize마다 재계산됨.
		pageBottoms: {},

		// 요청 페이지 관리 객체
		reqMap: {},

		convertServerUri : localSynap.properties.contextPath,
		offsetGetImageSize : 30,
		
		isPrintPage: function (pageNum) {
			return localSynap.startPage <= pageNum && pageNum <= localSynap.endPage
		},
		parsePdfXML: function(xml) {
			localSynap.infoXml = xml;

			if ($(xml).find('startPage').length > 0) {
				localSynap.startPage = parseInt($(xml).find('startPage').text());
			} else {
				localSynap.startPage = 1;
			}

			if ($(xml).find('endPage').length > 0) {
				localSynap.endPage = parseInt($(xml).find('endPage').text());
			} else {
				localSynap.endPage = parseInt($(xml).find('pdf_cnt').text());
			}
			if (localSynap.properties.entireWithPartialConv == true) {
				localSynap.pageSize = parseInt($(xml).find('pdf_cnt').text());
			} else {
				if (localSynap.endPage - localSynap.startPage + 1 >  parseInt($(xml).find('pdf_cnt').text()) ) {
					localSynap.pageSize = parseInt($(xml).find('pdf_cnt').text());	
				} else {
					localSynap.pageSize = localSynap.endPage - localSynap.startPage + 1;
				}
				
			}
			$(xml).find('pdf').each(function (index) {
				if (localSynap.properties.entireWithPartialConv == false) {
					pageNum = index + 1;
					if ( !member.isPrintPage(pageNum) ) {
						return;
					}
				}
				obj = {
					index: (parseInt($(this).find('id').text())),
					title: $(this).find('title').text(),
					path: encodeURI( $(this).find('path_html').text()),
					w: (parseInt($(this).find('w').text())),
					h: (parseInt($(this).find('h').text()))
				};
				member.rawObjList.push(obj);
				localSynap.objList.push(obj);
			});
		},
		
		parseImgData : function(key, index, dpi) {
			dpi = dpi || defaultDPI;
			
			// 이미지 사이즈는 초기에만 호출하고, 차후에 동적로딩에서 재보정 하든지 한다.
			var i = index;
			while(i < localSynap.pageSize) {
				member.getImageSize(key, i, dpi);
				i = i + member.offsetGetImageSize;
			}
		},
		
		isDynamicLoading: function(){
			if( pdf.INIT_IMAGE_LIMIT == LIMIT_NUMBER || localSynap.pageSize <pdf.INIT_IMAGE_LIMIT) {
				return false;
			}else{
				return true;
			}
		},

		// 이미지 요청하여 받아오는 함수
		getImagePath: function(pageIndex){
			var pageNum = 1 + pageIndex;
			if (localSynap.properties.isRenderServer && localSynap.isImageMode()) {
				return member.convertServerUri + localSynap.objList[pageIndex].path + '?dpi=M';
			} else if (localSynap.properties.entireWithPartialConv == true) {
				// 로딩화면을 띄운다.
				var img_path = localSynap.pathMap[pageIndex];
				if ( img_path === undefined ) {
					loadSpinner('contents_pdf_spinner');
					$.ajax({
						type: "GET",
						url: localSynap.getBasePath() +localSynap.properties.fileName + '/' + pageNum,
						async: false,
						dataType: "json",
						error: function (data) {
							//console.log('image path error');
						},
						success: function (data) {
							////console.log(data.img_path);
							if (data["return"] == "true") {
								img_path = localSynap.objList[pageIndex].path;
								localSynap.pathMap[pageIndex] = img_path;
							}
							removeSpinner();
						}
					});
				}
				// 로딩화면 해제
				return localSynap.getResultDir() +img_path;
			} else {
				return localSynap.getResultDir()+localSynap.objList[pageIndex].path;
		  }
		},
		getThumbPath: function(pageIndex){
			return localSynap.getImagePath(pageIndex);
		},
		getHighQualPath: function(pageIndex){
			if (localSynap.properties.isRenderServer && localSynap.isImageMode()) {
				return member.convertServerUri + localSynap.objList[pageIndex].path + '?dpi=H';
			} else if ( localSynap.properties.entireWithPartialConv == true ) {
				 return localSynap.getImagePath(pageIndex);
			} else {
				return localSynap.getResultDir()+localSynap.objList[pageIndex].path;
			}
		},
		
		convRequest: function (pageIndex, callback) {
			var url = localSynap.getBasePath() + localSynap.properties.requestContext;
			url = url.replace("${id}", localSynap.properties.fileName);
			url = url.replace("${pageNum}", 1+pageIndex);
			$.ajax({
				type: "GET",
				url: url,
				dataType: "json",
				cache: false,
				error: function (xhr, status, foo) {
					console.log(status + " what?? " + pageIndex);
				},
				success: function (data) {
					if (data["return"] == "true") {
						callback(pageIndex, data["path"]);
					}
				},
				complete: function(data) {
				}
			});
		},

		// PDF서버에서 사용하는 함수
		setImageSrc: function(tagIdPrefix, pageIndex, imagePath) {
			localSynap.pathMap[pageIndex] = imagePath;
			var elImg = document.getElementById(tagIdPrefix+(pageIndex));
			if (elImg!=null){
				if (elImg.src.indexOf(imagePath) < 0) {
					elImg.src = localSynap.getResultDir() + imagePath;
				}
			}
			if (BROWSER.isMobile() == false) {
				localSynap.resizePage(pageIndex);
			}
		},
		// PDF서버에서 사용하는 함수
		setImageSection: function(prefixId, pageIndex, callback){
			console.log(prefixId, pageIndex);
			var reqIndex = localSynap.properties.partConvCnt * parseInt( pageIndex / localSynap.properties.partConvCnt ) ;
			if (member.reqMap[reqIndex] == undefined) {
				member.reqMap[reqIndex] = false;
			} else if (member.reqMap[reqIndex] == true) {
				var lastPageNum = reqIndex+localSynap.properties.partConvCnt > localSynap.getPageSize() ? localSynap.getPageSize() : reqIndex + localSynap.properties.partConvCnt;
				for (var index = reqIndex; index < lastPageNum; ++index) {
					member.setImageSrc(prefixId, index, localSynap.objList[index].path);
				}
				callback && callback(pageIndex, localSynap.objList[index].path);
				return;
			} else if (member.reqMap[reqIndex] == false) {
				setTimeout(function () {
					member.setImageSection(prefixId, pageIndex);
				}, 500);
				return;
			}
			// 로딩화면을 띄운다.
			loadSpinner('contents_pdf_spinner');
			localSynap.spinnerCounter++;
			member.convRequest(reqIndex, function (pageIndex, srcPath) {
				member.reqMap[reqIndex] = true;
				// use Paths in XML instead of srcPath from Server
				var lastPageNum = pageIndex+localSynap.properties.partConvCnt > localSynap.getPageSize() ? localSynap.getPageSize() : pageIndex + localSynap.properties.partConvCnt;
				for (var index = pageIndex; index < lastPageNum; ++index) {
					member.setImageSrc(prefixId, index, localSynap.objList[index].path);
				}
				callback && callback(pageIndex, srcPath);

				--localSynap.spinnerCounter;
				if (localSynap.spinnerCounter == 0) {
					removeSpinner();
				}
			});
		},
		setPageBottoms : function () {
			//console.log('setPageBottoms() start');
			len = localSynap.pageSize;
			var $pageArea;
			for (var index = 0; index < len; ++index) {
				$pageArea = pdf.$pageAreas[index];
				member.pageBottoms[index] = member.$pdfFrame.scrollTop() + $pageArea.position().top + $pageArea.height();
			}
			//console.log('setPageBottoms() finish');
		},
		eventDesktopScroll : function (event) {
			len = localSynap.pageSize;
			if (enableScrollEvent) {
				$pdfFrame = member.$pdfFrame;
				for (var i=0; i<len; i++) {
					if ( member.pageBottoms[i] > $pdfFrame.scrollTop() + $pdfFrame.height()/2) {
						if ((i+1)!=localSynap.getCurrentPage()){
							pdf.changePaperPageNumber(i + 1);
							pdf.imageReload(i + 1, 'page', member.arrPageShowList);
							pdf.moveThumbPage(i + 1);
						}
						break;
					}
				}
			}
			enableScrollEvent = true;

			// 스크롤이 멈추면 텍스트를 출력한다.
			member.arrangePageTextAsync();
			localSynap.onScroll && localSynap.onScroll(e);
		},
		eventMobileScrollBase : function (event) {
			if (enableScrollEvent) {
				var pageChangeBaseHeight = member.$pdfFrame.height()/2;
				var pageIdx = localSynap.getCurrentPage() - 1;	// 스크롤 이전의 현재 페이지인덱스
				var pageObj = pdf.$pageAreas[pageIdx];
				var pageBottom = pageObj.position().top + pageObj.height();
				
				if ( localSynap.lastScrollTop < member.$pdfFrame.scrollTop() ) {
					// 아래로 스크롤 할 때
					// 스크롤 마지막은 끝페이지 넘버를 준다.
					if (document.getElementById("contents").scrollHeight == (member.$pdfFrame.scrollTop() + member.$pdfFrame.height())) {
						pageIdx = localSynap.pageSize - 1;
						
					} else {
						while ( pageBottom < pageChangeBaseHeight && pageIdx >= 0) {
							++pageIdx;
							pageObj = pdf.$pageAreas[pageIdx];
							pageBottom = pageObj.position().top + pageObj.height();
						}
					}
				} else {
					// 위로 스크롤 할 때
					if (member.$pdfFrame.scrollTop() == 0) {
						pageIdx = 0;
					} else {
						while ( pageBottom > pageChangeBaseHeight && pageIdx >= 0) {
							--pageIdx;
							if (pageIdx < 0) {
								break;
							}
							pageObj = pdf.$pageAreas[pageIdx];
							pageBottom = pageObj.position().top + pageObj.height();
							
						}
						++pageIdx;
					}
				}
				pdf.changePaperPageNumber(pageIdx + 1);
				pdf.imageReload(pageIdx + 1, 'page', member.arrPageShowList);
			}
			enableScrollEvent = true;
			localSynap.lastScrollTop = member.$pdfFrame.scrollTop();
			localSynap.onScroll && localSynap.onScroll(e);
			
		},
		eventMobileScrollIOS : function (evnet) {
			if (enableScrollEvent) {
				// 페이지 번호는 실시간으로 업데이트
				var pageChangeBaseHeight = $(window).scrollTop() + $(window).height()/2;
				var pageIdx = localSynap.getCurrentPage() - 1;	// 스크롤 이전의 현재 페이지인덱스
				if (pageIdx > localSynap.objList.length - 1) {
					pageIdx = localSynap.objList.length - 1;
				}
				var pageObj = pdf.$pageAreas[pageIdx];
				var pageBottom = pageObj.position().top + pageObj.height();	// 40은 헤더영역
				var winScrollTop = $(window).scrollTop();
				// 스크롤 성능 개선
				if ( localSynap.lastScrollTop < winScrollTop ) {
					// 아래로 스크롤 할 때
					var winHeight = $(window).height();
					while( winScrollTop + winHeight> pageObj.position().top + pageObj.height()) {
						++pageIdx;
						if (pageIdx + 1 > localSynap.objList.length) {
							pageIdx = localSynap.objList.length;
							break;
						}
						pageObj = pdf.$pageAreas[pageIdx];
					}
					--pageIdx;
				} else {
					// 위로 스크롤 할 때
					while( winScrollTop  < pageObj.position().top  ) {
						--pageIdx;
						if ( pageIdx < 0 ) {
							break;
						}
						pageObj = pdf.$pageAreas[pageIdx];
					}
					++pageIdx;
				}
				// 스크롤 시 첫페이지는 예외처리 해준다.
				if ( winScrollTop < 20 || pageIdx < 0 ) {	  // 빈틈 20
					pageIdx = 0;
				}
				pdf.changePaperPageNumber(pageIdx + 1);
				pdf.imageReload(pageIdx + 1, 'page', member.arrPageShowList);
			}
			enableScrollEvent = true;
			localSynap.lastScrollTop = $(window).scrollTop();
		},
		getImageSize : function (key, pageIdx, dpi) {
			pageIdx = (typeof pageIdx !== "undefined") ? pageIdx : 0;
			dpi = (typeof dpi !== "undefined") ? dpi : defaultDPI;
			var obj = {
				index: 0,
				path: "",
				w: 0,
				h: 0
			};
			var url = member.convertServerUri + '/dimension/' + key + '/' + pageIdx +'-'+ member.offsetGetImageSize;
			$.ajax({
				type: "GET",
				url: url,
				async: false,
				dataType: "json",
				error: function(data){
					//alert('error')
				},
				complete : function(data){
					//alert('complate')
				},
				success:function(data) {
					// 썸네일 영역을 뺀 부분을 화면표시 너비로 잡아준다.
					var thumbWidth = 0;
					if ( $('thumbnail').length > 0 ) {
						thumbWidth = parseInt($('#thumbnail').css('width'));
					}
					var winWidth = $(window).width() - thumbWidth - 20;
					// 헤더영역보다 조금 더 여유있게 화면표시 높이를 잡아준다.
					var headerHeight = 0;
					if ( $('#header').length > 0 ) {
						headerHeight = parseInt($('#header').css('height'));
					}
					var winHeight = $(window).height() - headerHeight * 2;
					$.each(data, function (idx, elem) {
						if (typeof member.rawObjList[elem.p] != "undefined") return;
						
						member.rawObjList[elem.p] = {
							index: elem.p,
							path: '/thumbnail/'+key+'/'+ elem.p,
							w: elem.w,
							h: elem.h
						};
						// 가로가 긴 문서는 화면폭에 맞춰줌
						var objHeight = elem.h;
						var objWidth = elem.w;
						var heightPerWidth = objHeight / objWidth;
						var width = winWidth > objWidth ? objWidth : winWidth;
						var height = width == elem.w ? objHeight : width * heightPerWidth;
						if (height > winHeight) {
							height = winHeight;
							width = height / heightPerWidth;
						}

						obj = {
							index: elem.p,
							path: '/thumbnail/'+key+'/'+ elem.p,
							w: width,
							h: height
						};
						member.rawObjList[elem.p] = obj;
						localSynap.objList[elem.p] = obj;
					});
				}
			});
		},
		
		// 이미지 태그를 생성한다. src는 빈칸으로 생성하고, 동적로딩에서 path를 넣어준다.
		createPageElement: function (pageIndex) {
			// 초기에는 기준갯수만 이미지 로딩
			if( !member.isDynamicLoading() || pdf.INIT_IMAGE_LIMIT>pageIndex) { // 동적로딩 설정이 되어 있지않거나 동적로딩 갯수보다 낮은 순번일때
				member.arrPageShowList.push(pageIndex);
			}
			
			var failImagePath = 'image/common/img_loading.png';
			var html = '<img id="page' + pageIndex +'" name="'+pageIndex+'" src="" style="background-position: 50% 50%; background-repeat: no-repeat; background-image: url('
				+ failImagePath  + ');" unselectable="on"  onload="onLoadImg('+ pageIndex +')" title="page'+ (pageIndex + 1) +'" onerror="image_error(this)" alt="page'+ (pageIndex + 1) +'" />';
			
			return html;
		},
		
		createThumbElement: function (pageIndex) {
			var curWidth = localSynap.properties.thumbnailWidth;
			var thumbHeight = parseInt((curWidth/localSynap.objList[pageIndex].w) * localSynap.objList[pageIndex].h);
			
			// 초기에는 기준갯수만 이미지 로딩
			if( pdf.INIT_IMAGE_LIMIT == LIMIT_NUMBER || pdf.INIT_IMAGE_LIMIT>pageIndex) { // 동적로딩 설정이 되어 있지않거나 동적로딩 갯수보다 낮은 순번일때
				member.arrThumbShowList.push(pageIndex);
			}
			
			var failImagePath = 'image/common/thumb_loading.png';
			var html = '<img id="thumb' + pageIndex +'" name="'+pageIndex
				+'" src="" style="height:' + thumbHeight + 'px;width:' + curWidth
				+ '; background-position: 50% 50%; background-repeat: no-repeat; background-image: url(' + failImagePath + ')" unselectable="on" onerror="image_thum_error(this)" title="page'
				+ (pageIndex + 1) +'" alt="thumb'+ (pageIndex + 1) +'" onload="//console.log(' + pageIndex + ')"/>';
			
			return html;
		},

		// 약간의 딜레이와 함께 텍스트를 출력한다.
		// 딜레이시간 내 재실행은 무시한다.
		arrangePageTextAsync: function () {
			if (localSynap.highlightQueue == null) {
				timeout = 2000;
			} else {
				timeout = 100;
			}
			if (member.scrollTimer !== null) {
				clearTimeout(member.scrollTimer);
			}
			member.scrollTimer = setTimeout(function () {
				//console.log('append timeout : ', timeout);
				// 이외 부분은 꾸준히 청소해준다.
				var currentIdx = localSynap.getCurrentPage() -1;
				if (localSynap.properties.usePdfText) {
					if ( localSynap.isImageMode() ) {
						localSynap.parseTextXmlWithKey(currentIdx);
					} else {
						localSynap.parseTextXmlFromFile(localSynap.properties.fileName, currentIdx);
					}
				}
				cleanSpans(currentIdx);
			}, timeout)
		},

		initEvent: function() {
			if (BROWSER.isMobile()){
				member.initEventMobile();
			} else {
				member.initEventDesktop();
			}
			
			if (localSynap.hasPdfFrame()){
				if ( BROWSER.isMobile() ) {
					localSynap.lastScrollTop = LIMIT_NUMBER * 1000;
					if ( BROWSER.MOBILE.isIOS() ) {
						$(document).scroll(member.eventMobileScrollIOS);
					} else {
						member.$pdfFrame.scroll(member.eventMobileScrollBase);
					}
				} else {
					// 데스크탑 스크롤
					member.$pdfFrame.scroll(member.eventDesktopScroll);
				}
			}
		},

		initEventDesktop: function() {
			$(document).delegate('#searchBoxBtn', 'click', function () {
				$('#searchBox').toggleClass('activePop');
				$('#searchText').focus();
			});
			$(document).delegate('#searchText', 'focus', function (event) {
				preventKeydown = true;
				//console.log('preventkeydown', true);
			});
			$(document).delegate('#searchText', 'focusout', function (event) {
				preventKeydown = false;
				//console.log('preventkeydown', false);
			});
			$(document).delegate('#searchText', 'keydown', function (event) {
				if(event.keyCode==13) {
					prev = false;
					if (event.shiftKey) {
						prev = true;
					}
					//console.log("search enter", prev);
					pdfSearch.search($('#searchText').val(), prev);
				}
			});
			$(document).delegate('#prevSearchTextButton', 'click', function () {
				try{
					var targetText = $('#searchText').val();
					pdfSearch.search(targetText, true);
				} catch(e) {
					//console.log("no search result");
				}
			});
			$(document).delegate('#nextSearchTextButton', 'click', function () {
				try{
					var targetText = $('#searchText').val();
					pdfSearch.search(targetText, false);
				} catch(e) {
					//console.log("no search result");
				}
			});
			$(document).delegate('#slideJumpButton', 'click', function(){
				try{
					var pageNum = parseInt($('#inputPageNumber').val());
					if( pageNum > localSynap.pageSize || pageNum < 0 || isNaN(pageNum) )
						throw new Error("wrong Number");
					pdf.movePage(pageNum);
				}catch(e){
					$('#inputPageNumber').val( localSynap.getCurrentPage() );
				}
			});

			$(document).delegate('#inputPageNumber', 'keypress', function(e){
				if (e.which==13){
					$('#slideJumpButton').click();
				}
			});

			$('a.closeBtn').on('click', function(e) {
				e.preventDefault();
				$('#thumbnail').animate({ left: -$('#thumbnail').width() },
					function() {
						$(window).resize();
						// $('#leftPanel_hidden').toggle(true);
					});
				localSynap.leftPanelShow = !localSynap.leftPanelShow;
			});
			$('a.openThumbnail').on('click', function(e){
				e.preventDefault();
				
				$('#thumbnail').animate({ left: 0 },
					function() {
						$(window).resize();
						// $('#leftPanel_hidden').toggle(false);
					});
				localSynap.leftPanelShow = !localSynap.leftPanelShow;
			});

			$(document).delegate('#documentScale', 'click', function(e){
				if (enableZoomEvent){
					enableZoomEvent = false;
					var cur_ratio = RATIO_NUMBERS.getRatio();
					var base_ratio = RATIO_NUMBERS.getRatio();
					var btn_top = $('#documentScale').offset().top;  // bar 상대top
					var btn_height = $('#documentScale').height(); // bar Height
					var evt = e.clientY || window.event.y;
					var mouse_y = evt - btn_top; // bar 영역내 클릭 위치
					var tab_range = parseInt(btn_height / 3);
					
					// 아래는 코드정리가 필요하다.
					var new_ratio = RATIO_NUMBERS.getValueOfZoom(0);
					var bar3_top = btn_top;
					if (mouse_y<tab_range){
						new_ratio = RATIO_NUMBERS.getValueOfZoom(2);
					} else if (mouse_y<(tab_range*2)) {
						new_ratio = RATIO_NUMBERS.getValueOfZoom(1);
						bar3_top += parseInt(btn_height/2);
					}else{
						bar3_top += btn_height;
					}
					bar3_top = bar3_top - parseInt($('.bar3').height()/2);
					$('.state').text(new_ratio+'X');
					
					while (cur_ratio != new_ratio) 
					{
						if (cur_ratio<new_ratio){
							cur_ratio = RATIO_NUMBERS.increaseRatio();
						}else if (cur_ratio>new_ratio){
							cur_ratio = RATIO_NUMBERS.reduceRatio();
						}
					}
					member.zoom(base_ratio, cur_ratio);
					
					$('.bar3').offset({top:bar3_top});
					enableZoomEvent = true;
				}
			});
			
			$(document).keydown(function(e) {
				if (e.which == 38 || e.which == 40) {
					e.preventDefault();
				} else if (e.which == 37) {
					//e.preventDefault();
				} else if (e.which == 39) {
					//e.preventDefault();
				} else if (e.which == 33 || e.which == 34) {
					e.preventDefault();
				}

				// Ctrl + F
				if (e.ctrlKey && e.keyCode == 70 ) {
					e.preventDefault();
					$box = $('#searchBox');
					$input = $('#searchText');
					if ($box.hasClass('activePop') == true) {
						if ($input.get(0) == document.activeElement) {
							$box.toggleClass('activePop');
						}
					} else {
						$box.toggleClass('activePop');
					}
					$input.focus();
				}
			});
			// 썸네일 영역이 있으면 변수값을 true로 초기화 한다.
			if (localSynap.hasIndexFrame()){
				localSynap.leftPanelShow = true;
			}else{
				localSynap.leftPanelShow = false;
			}

			if (localSynap.hasIndexFrame()){
				member.$idxFrame.parent().scroll(function(e) {
					if (enableScrollEvent) {
						var idxScrollFrame = member.$idxFrame.parent();
						var pageChangeBaseHeight = idxScrollFrame.height()/2;
						for (var i=0; i<localSynap.pageSize; i++) {
							var pageBottom = $("#thumb-area" + i).offset().top + $("#thumb-area" + i).height();
							if ( pageBottom > pageChangeBaseHeight) {
								// 동적로딩을 그린다.
								pdf.imageReload(i, 'thumb', member.arrThumbShowList);
								break;
							}
						}
					}
					enableScrollEvent = true;
				});
			}
		},
		
		initEventMobile: function() {
			/*
			$('.navPrev').click(function(e){
					e.preventDefault();
					localSynap.movePrev();
				}
			);
			$('.navNext').click(function(e){
					e.preventDefault();
					localSynap.moveNext();
				}
			);
			*/
			$('.paging').click(function(e) {
				enableScrollEvent = false;
			});
			
			$(window).on("orientationchange",function(){
				if ( BROWSER.MOBILE.isGalaxyNote3() ) {
					$('#wrap').css('height', $(window).height() - $('#header').height());
				}
				// 안드로이드 ICS는 landscape에만 변환을 해준다.(사실 portrait로 돌아갈때 objList가 왜 크게 변형되는지 모르겠음..)
				if (BROWSER.VERSION.isAndroidICS()) {
					if(window.orientation == 0) // Portrait
					{
					}
					else // Landscape
					{
						// 썸네일 영역을 뺀 부분을 화면표시 너비로 잡아준다.
						var thumbWidth = 0;
						if ( $('thumbnail').length > 0 ) {
							thumbWidth = parseInt($('#thumbnail').css('width'));
						}
						var winWidth = $(window).width() - thumbWidth - 20;
						// 헤더영역보다 조금 더 여유있게 화면표시 높이를 잡아준다.
						var headerHeight = 0;
						if ( $('#header').length > 0 ) {
							headerHeight = parseInt($('#header').css('height'));
						}
						var winHeight = $(window).height() - headerHeight * 2;
						$.each(member.rawObjList, function (idx, elem) {
							// 가로가 긴 문서는 화면폭에 맞춰줌
							var objHeight = elem.h;
							var objWidth = elem.w;
							var heightPerWidth = objHeight / objWidth;
							var width = winWidth > objWidth ? objWidth : winWidth;
							var height = width == elem.w ? objHeight : width * heightPerWidth;
							if (height > winHeight) {
								height = winHeight;
								width = height / heightPerWidth;
							}
							
							obj = {
								index: elem.index,
								path: '/thumbnail/'+localSynap.jobId+'/'+ elem.index,
								w: width,
								h: height
							};
							localSynap.objList[elem.index] = obj;
							localSynap.resizePage(idx);
						});
					}
				}
			});
		},
		
		zoom: function(oldRatio, newRatio) {
			// horizontal scroll
			/******************************************************************/
			var contentWidth = member.$pdfFrame.width();
			
			var prevScrollW = member.$pdfFrame.get(0).scrollWidth;
			var prevScrollL = member.$pdfFrame.scrollLeft();
			var scrollLeftRatio = (prevScrollL / (prevScrollW - contentWidth));
			/******************************************************************/
			
			var offsetTop = pdf.$pageAreas[localSynap.getCurrentPage() - 1].position().top * newRatio / oldRatio;
			member.resizePageAll();
			
			// vertical scroll
			/******************************************************************/
			localSynap.movePage(localSynap.getCurrentPage());
			member.$pdfFrame.scrollTop(member.$pdfFrame.scrollTop() - offsetTop + 20);		// 20은 자동 마진영역
			
			/******************************************************************/
			
			// horizontal scroll
			/******************************************************************/
			var newScrollW = member.$pdfFrame.get(0).scrollWidth;
			var newScrollLeft = (newScrollW - contentWidth) * scrollLeftRatio;
			member.$pdfFrame.scrollLeft(newScrollLeft);
			/******************************************************************/
		},
		
		getThumbnailWidth: function() {
			// TODO: 초기화 접근자를 멤버변수로 들고있는 것이 좋을 것 같다.
			if (localSynap.hasIndexFrame() && $('#thumbnail').length>0){
				return $('#thumbnail').get(0).offsetWidth;
			}else{
				return 0;
			}
		},
		resizeOnClosingSelect: function() {
			var $contentWrap = $('#contents');
			// landscape / portrait 회전 과정에서, contents 자체 높이로 계산하니, 높이가 변경되지 않아, 이 함수가 호출될때 마다 새로 계산함.
			var height = $(window).height() - $('#header').height();
			$contentWrap.height(height-1);
			$contentWrap.height(height);
		},
		// 이미지의 resize 함수(확대/축소에 따른)
		resizePageAll: function() {
			var len = localSynap.objList.length;
			loadSpinner('contents_pdf_spinner');
			//console.log('pageAll start');
			for (var index = 0; index < len; ++index) {
				localSynap.resizePage(index);
			}
			removeSpinner();
			member.setPageBottoms();
			//console.log('pageAll end');
		},

		resize: function() {
			if (BROWSER.isMobile()){
				// 모바일에서는 화면맞춤을 하므로 resize하지 않는다.
			}else{
				member.resizeDesktop();
			}
		},
		
		resizeDesktop: function() {
			setResizeHeaderTitle(); // #DEFECT-2580 title resize를 위해 호출
			
			if( localSynap.leftPanelShow === true )
				member.$pdfFrame.css('left', member.getThumbnailWidth()); 
			else
				member.$pdfFrame.css('left', 0);
			
			member.resizePageAll();
		}
	};

	var pdf = {
		curPage: 1,
		FILENAME: encodeURI(localSynap.getFileName()),
		pageSize: 0,  // api.js 에서도 pageSize를 설정해주는데, 여기에서 덮어씌우는건 아닌가?
		ratio: 1,
		pxToPtRatio: undefined,
		scrollTopValue: 0,
		loadedImgCount: 0,

		// 배율이 적용된 페이지정보 리스트. 주로 사용되는 객체
		objList: [],
		INIT_IMAGE_LIMIT: 10, // 동적로딩 갯수. LIMIT_NUMBER로 설정하면 동적로딩을 하지 않는다.
		INIT_MAKE_LIMIT: 10, // 초기화시 로딩 이미지 갯수
		spinnerCounter: 0, // 로딩 스피너 카운터
		searchMeta: {},
		fullText: {},
		highlightQueue: null,
		// jQuery 객체를 담는 변수(성능관련)
		$pageAreas: [],
		$pages: [],

		// common.js 에서 호출하는 함수들 (제거금지)

		thumbnailScrollPositionFix: function () {
			// 전체화면 해제 시 썸네일 스크롤을 본문과 동기화한다.
			//console.log('thumbnailscrollPositionFix');
			pdf.moveThumbPage(localSynap.curPage);
		},

		// common.js 영역 종료

		hasIndexFrame: function() {
			if (typeof member.$idxFrame != "undefined") {
				return member.$idxFrame.length>0; 
			} else {
				return false;
			}
		},

		hasPdfFrame: function() {
			if (typeof member.$pdfFrame != "undefined") {
				return member.$pdfFrame.length>0; 
			} else {
				return false;
			}
		},

		parseImgXml: function(xml) {
			localSynap.infoXml = xml;
			localSynap.pageSize = parseInt($(xml).find('image_cnt').text());
			$(xml).find('image').each(function () {
				obj = {
					index: (parseInt($(this).find('id').text())),
					path: encodeURI( $(this).find('path_image').text()),
					w: (parseInt($(this).find('width').text())),
					h: (parseInt($(this).find('height').text()))
				};
				member.rawObjList.push(obj);
				localSynap.objList.push(obj);
			});
		},
		prepareFullText: function (index) {
			var textObj = localSynap.searchMeta[index];
			if (localSynap.fullText[index] != undefined) {
				return;
			}
			$(textObj).each(function (textIndex, elem) {
				if (typeof localSynap.fullText[index] === "undefined") {
					localSynap.fullText[index] = elem.text;
				} else {
					localSynap.fullText[index] += elem.text;
				}
			});
		},	
		parseTextXmlWithKey: function (pageIdx) {
			var key = localSynap.jobId;
			var targetUrl = member.convertServerUri +'/thumbnailxml/'+key+'/'+pageIdx+'?dpi='+defaultDPI;
			pdfSearch.loadTextXmlAsync(targetUrl, pageIdx);
		},

		parseTextXmlFromFile: function (fileName, index) {
			if (localSynap.properties.fileType == 'pdf') {
				var pageNum = parseInt(index) + 1;
				var xmlFile = fileName + '_' + pageNum + '.xml';
				if (localSynap.properties.entireWithPartialConv == true) {
					targetUrl = localSynap.getResultDir() + fileName + '.files/' + xmlFile;
				} else {
					targetUrl = localSynap.getResultDir() + fileName + '.files/' + xmlFile;
				}
				pdfSearch.loadTextXmlAsync(targetUrl, index);
			} else {
				var padding = '0000';
				// 이미지변환파일은 개별폴더가 아닌 대상폴더에 모두 생성된다.
				var pageNum = parseInt(index)+1;
				var padNo = padding.substring(0, padding.length -(pageNum+"").length) + pageNum;
				file = localSynap.getResultDir() + fileName + '_' + padNo + '.xml';
				pdfSearch.loadTextXmlAsync(file, index);
			}
		},
		// 텍스트 검색 실행 함수
		search : function (text, isPrev) {
			pdfSearch.search(text, isPrev);
		},
		// span태그에 highlight 처리를 한다.
		setHighlight : function (pageIdx, textIdx, length, remove) {
			var $pageElem = pdf.$pageAreas[pageIdx].find('span');
			if ($pageElem.length <= 0) return;
			// 모든 span태그를 돌면서
			for (var index = 0; index < $pageElem.length; ++index) {
				// 범위 밖이면 다음으로 패스
				if (index < textIdx || index >= textIdx +length) {
					continue;
				}

				if (index >= pdfSearch.findTextIdx && index < pdfSearch.findTextIdx + length) {
					$pageElem[index].style.backgroundColor = 'red';
				} else {
					$pageElem[index].style.backgroundColor = 'blue';
				}
				if (BROWSER.VERSION.IE() <= 9) {
					$pageElem[index].style.filter = 'alpha(opacity=50)';
				} else {
					$pageElem[index].style.opacity = 0.5;
				}
			}
			return;
		},
		// jQuery객체를 다시 스트링으로 바꾸는게.. 과연 옳은일인가?
		updateHighlightSpan: function (pageIndex, $spans) {
 			// 검색상태이면 하이라이트 처리한다.
			var highlight_obj = localSynap.highlightQueue;
			var txtlen = highlight_obj.text.length;
			var offsetList = highlight_obj.list[pageIndex];
			if (offsetList == undefined) {
				return $spans;
			}
			for (var hlIndex = 0; hlIndex < offsetList.length; ++hlIndex) {
				var startIdx = offsetList[hlIndex];
				var endIdx = startIdx + txtlen;

				for (var spanIdx = startIdx; spanIdx < endIdx; ++ spanIdx) {
					$span = $spans[spanIdx];
 					// 포커스된 부분만 다른색으로 표현한다.
					if (pageIndex == pdfSearch.findPageIdx
						&& spanIdx >= pdfSearch.findTextIdx
						&& spanIdx < pdfSearch.findTextIdx + txtlen) {
						$span.style.backgroundColor = 'red';
					} else {
						$span.style.backgroundColor = 'blue';
					}

					$span.style.opacity = 0.5;
				}
			}
			return $spans;
		},
		updateHighlightSpan2: function (pageIndex) {
			//console.log("updateHighlightSpan2 () start");
 			// 검색상태이면 하이라이트 처리한다.
			var highlight_obj = localSynap.highlightQueue;
			var txtlen = highlight_obj.text.length;
			var offsetList = highlight_obj.list[pageIndex];
			if (offsetList == undefined) {
				return;
			}
			var len = offsetList.length;
			$spans = localSynap.$pageAreas[pageIndex].find('span');
			for (var offsetIdx = 0; offsetIdx < len; ++offsetIdx) {
				hlOffsetStart = offsetList[offsetIdx];
				hlOffsetEnd = hlOffsetStart + txtlen;
				for (var spanIdx = hlOffsetStart ; spanIdx < hlOffsetEnd; ++spanIdx) {
					$span = $spans[spanIdx];
					// 포커스된 부분만 다른색으로 표현한다.
					if (pageIndex == pdfSearch.findPageIdx
						&& spanIdx >= pdfSearch.findTextIdx
						&& spanIdx < pdfSearch.findTextIdx + txtlen) {
						$span.style.backgroundColor = 'red';
					} else {
						$span.style.backgroundColor = 'blue';
					}

					$span.style.filter = 'alpha(opacity=50)';
				}
			}
			//console.log("updateHighlightSpan2 () finish");
			return ;
		},
		getText: function (pageIndex) {
			// RenderServer에서 text를 image보다 먼저 줄 수 없을 때를 방지하기 위한 코드. 추후에 개선되면 삭제하자.
			if (localSynap.properties.isRenderServer && localSynap.isImageMode()) {
				if (pageIndex in member.arrVisitPageIdx == false) {
					return "";
				}
			}

			var textSpan = pdfSearch.getSpanString(pageIndex, localSynap.searchMeta);

			var $textSpan = $(textSpan);
			if (localSynap.highlightQueue != null) {
				pdfSearch.updateHighlightQueue(pageIndex, localSynap.highlightQueue.text);
				pdf.updateHighlightSpan(pageIndex, $textSpan);
			}
			return $textSpan;
		},
		appendSpanTags: function (pageIndex) {
			pdfSearch.getSpanObject(pageIndex, localSynap.searchMeta);
			if (localSynap.highlightQueue != null) {
				pdfSearch.updateHighlightQueue(pageIndex, localSynap.highlightQueue.text);
				textSpan = pdf.updateHighlightSpan2(pageIndex);
			}
		},
		// span태그를 append하는 함수
		appendText: function (xmlObj, append_text_async) {
				// span태그를 위한 정보를 들고있는다.
				if (localSynap.searchMeta[xmlObj.index] == undefined) {
					localSynap.searchMeta[xmlObj.index] = xmlObj.content;
				}
				append_text_async(xmlObj);
		},
		getImagePath: function (pageIdx) {
			if (pageIdx in member.arrVisitPageIdx == false) {
				member.arrVisitPageIdx.push(pageIdx);
			}
			return member.getImagePath(pageIdx);
		},
		getThumbPath: function (pageIdx) {
			return member.getThumbPath(pageIdx);
		},
		setImage: function (tagIdPrefix, pageIndex, callback) {
			member.setImageSection(tagIdPrefix, pageIndex, callback);
		},
		
		// 썸네일 전체에 대한 div를 생성한다.
		createThumbDiv: function () {
			//console.log("createThumbDiv()1 start");
			if (typeof member.$idxFrame != "undefined") {
				starterLen = 20;
				len = localSynap.pageSize;
				objList = localSynap.objList;
				curWidth = localSynap.properties.thumbnailWidth;
				for (var pageIndex = 0; pageIndex < len; ++pageIndex) {
					var html = '<div class="imgBox"  style="height:'
						+ parseInt((curWidth/objList[pageIndex].w) * objList[pageIndex].h) + 'px; width:'+curWidth
						+'px;"><em class="thum_num">' + (1+pageIndex) + '</em><div id="thumb-area' + pageIndex
						+ '" onclick="enableScrollEvent=false; localSynap.movePage(' + (1+pageIndex) + ');">'
						+ '</div>';
					member.$idxFrame.append(html);
				}
			}
			//console.log("createThumbDiv()1 finish");
		},
		
		createContentDiv: function (pageIndex) {
			var objList = localSynap.objList;
			html = '<div id="page-area' + pageIndex + '" class="page-element" ';
			if (BROWSER.isMobile()) {
				html += 'style="width:100%;'
				+ 'margin-left:auto; margin-right:auto; position:relative;">'
				+ '<img id="page' + pageIndex +'" name="'+pageIndex+'" src=""'
				+ ' style="width:100%; background-position: 50% 50%; background-repeat: no-repeat;';

			} else {
				html += 'style="width:'+objList[pageIndex].w+'px;height:'+objList[pageIndex].h+'px;'
				+ 'margin-left:auto; margin-right:auto; position:relative;">'
				+ '<img id="page' + pageIndex +'" name="'+pageIndex+'" src=""'
				+ ' style="background-position: 50% 50%; background-repeat: no-repeat;';
			}
			html += ' background-image: url(\'image/common/img_loading.png\');" unselectable="on"'
				+ ' onload="onLoadImg('+ pageIndex +')" title="page'+ (pageIndex + 1)
				+ '" onerror="image_error(this)" alt="page'+ (pageIndex + 1) +'" />'
				+ '</div>';
			$html = $(html);
			localSynap.$pageAreas.push($html.appendTo(member.$pdfFrame));
			$page = $html.find('img');
			if (pageIndex < pdf.INIT_MAKE_LIMIT) {
				if (localSynap.properties.entireWithPartialConv == true) {
					localSynap.setImage("page", pageIndex);
				} else {
					$page.attr('src', member.getImagePath(pageIndex));
				}
			}
			localSynap.$pages.push($page);
		},
		// 페이지 전체에 대한 div를 생성한다.
		createContentDivs: function () {
			var len = localSynap.pageSize;
			var starterLen = 10;
			if (starterLen > localSynap.pageSize) {
				starterLen = localSynap.pageSize;
			}
			for (var pageIndex = 0; pageIndex < starterLen; ++pageIndex) {
				localSynap.createContentDiv(pageIndex);
				
			}
			setTimeout(function (pageInde) {
				for (var pageInde = starterLen; pageInde < len; ++pageInde) {
					localSynap.createContentDiv(pageInde);

				}
				member.setPageBottoms();

				// iOS계열은 height를 주어야한다.
				if ( BROWSER.MOBILE.isIOS() ) {
					member.$pdfFrame.css('height', document.getElementById('contents').scrollHeight);
				}
			}, 1000);
		},
		
		// img를 INIT_IMAGE_LIMIT 갯수만큼만 만든다.
		loadThumbImage: function (startPageNum, endPageNum) {
			for (var i=startPageNum; i<endPageNum; i++) {
				//member.$idxFrame.append(member.createThumbElement(i));
				$('#thumb-area'+i).append(member.createThumbElement(i));
				if (i < pdf.INIT_MAKE_LIMIT) {
					if (localSynap.properties.entireWithPartialConv == true) {
						localSynap.setImage("thumb", i);
					} else {
						$('#thumb'+i).attr('src', member.getThumbPath(i));
					}
				}
			}
			if (endPageNum < localSynap.pageSize){ // 아직 읽을게 남았으면
				var s = endPageNum;
				var e = s + pdf.INIT_MAKE_LIMIT;
				if (e>localSynap.pageSize){
					e = localSynap.pageSize;
				}
				setTimeout('localSynap.loadThumbImage('+s+','+e+');', 0);
			}
		},
		
		changePaperPageNumber: function (pageNum) {
			if (BROWSER.isMobile()){
				$('.paging option').eq((pageNum-1)).prop('selected', true);
			}else{
				$('#inputPageNumber').val(pageNum);
				$("#totalPageNumber").text(localSynap.pageSize);
				
				$('#documentList > *').removeClass('thumbnailSel');
				$('#documentList > *:nth-child(' + (pageNum) + ')').addClass('thumbnailSel');
			}
			localSynap.curPage = pageNum;
		},
		// 범위 외 페이지는 이미지를 숨긴다.
		// TODO: display:none 과 src제거의 성능비교가 필요함
		refreshShowList: function (objName, startIdx, endIdx, arrShowList) {
			while(arrShowList.length > 0) {
				var idx = arrShowList.shift();
				if (idx >= startIdx && idx <=endIdx){
					// 이미 보여지고 있으니 삭제하지 않고 배열에서만 뺀다.
				}else{
					document.getElementById(objName+(idx)).src = "";
				}
			}
		},
		// 동적로딩
		// + img태그의 src를 삽입한다.
		imageReload: function (pageNum, objName, arrShowList) {
			//console.log('imageReload()', pageNum);
			if (pageNum == localSynap.getCurrentPage()) {
				// zoomInfo가 다르지 않으면 그냥 종료한다.

				// 다르다면 배율을 조정해야 한다.

				//console.log('page same');
			}
			var pageIndex = pageNum-1;
			var startIdx = parseInt(pageNum-(pdf.INIT_IMAGE_LIMIT/2)); // 기준의 절반 앞부터
			startIdx = startIdx < 0 ? 0: startIdx;

			var endIdx = startIdx + pdf.INIT_IMAGE_LIMIT - 1;
			endIdx = endIdx >= localSynap.pageSize ? localSynap.pageSize-1 : endIdx;

			/*
			// 검색중이면서, 방문한적 없는 페이지에서는 미리 검색을 수행하여 하이라이트정보를 모은다.
			if ( localSynap.highlightQueue.length && localSynap.highlightQueue[0].list[pageIndex] == undefined) {
				// 결과값은 필요없음. 내부에서 highlightQueue만 완성시키면 됨.
				pdfSearch.loadPageText(pageNum, function (){
					pdfSearch.updateHighlightQueue(pageIndex, pdfSearch.curSearchText);
				});
			}
			*/
			// 텍스트출력큐에 넣는다.
			pdfSearch.textOutputQueue.push(pageIndex);

			// 표시범위 가장자리를 벗어났을 때에만 다음 루틴으로 넘어간다.
			// 썸네일은 그냥 페이지마다 수행한다.
			if (objName == "thumb") {
				//console.log("imageReload() thumb go", pageNum);
			} else {
				if (arrShowList[1] < pageIndex && pageIndex < arrShowList[arrShowList.length - 2]) {
					//console.log('imageReload() pass ', pageNum);
					return;
				}
			}

			pdf.refreshShowList(objName, startIdx, endIdx, arrShowList);

			if (localSynap.properties.entireWithPartialConv == true) {
				for( var i = startIdx; i <= endIdx; ++i ){
					// 재변환 적용
					localSynap.setImage(objName, i);
					arrShowList.push(i);
				}
			} else {
				isBaseZoom = RATIO_NUMBERS.getRatio() == RATIO_NUMBERS.getValueOfZoom(0) ? true : false;
				for( var i = startIdx; i <= endIdx; ++i ){
					elImg = document.getElementById(objName+(i));
					// 재변환 적용
					imgpath = "";
					if (objName == "thumb") {
						imgpath = member.getThumbPath(i);
					} else {
						if (isBaseZoom) {
							imgpath = member.getImagePath(i);
						} else {
							imgpath = member.getHighQualPath(i);
						}
					};
					if (elImg!=null){
						if (elImg.src.indexOf(imgpath) < 0) {
							elImg.src = imgpath;
							//console.log('imgpath injection', i);
						}
						//중앙정렬이 안되어서 주석처리 elImg.style.display = 'block';
						arrShowList.push(i);
					}
					if (BROWSER.isMobile() == false) {
						localSynap.resizePage(i);
					}
				}
			}
			//console.log(arrShowList)
			//console.log('imageReload() finish ', pageNum);
		},

		// 1에서 시작
		movePrev: function() {
			if (localSynap.getCurrentPage() < 1) {
				return;
			} else {
				try{
					pdf.movePage(localSynap.getCurrentPage() - 1);
				}catch( e ){
					
				}
			}
		},

		moveNext: function() {
			if (localSynap.getCurrentPage() >= localSynap.pageSize) {
				return;
			} else {
				try{
					pdf.movePage(localSynap.curPage + 1);
				}catch( e ){
					
				}
			}
		},

		// 본문 화면이 해당 페이지로 이동한다.
		// 1부터 시작
		movePage: function (pageNum) {
			//console.log("movePage() start", pageNum);
			if (!(pageNum > 0 && pageNum <= localSynap.pageSize)){
				throw new Error("wrong number");
			}
			var contPos = $("#page0").parent().position();
			var prevPos =  $("#page" + (pageNum - 1)).parent().position();
			enableScrollEvent = false;
			pdf.scrollTopValue = parseInt( prevPos.top) - contPos.top;
			
			if ( BROWSER.MOBILE.isIOS() ) {
				$(window).scrollTop(pdf.scrollTopValue);
			} else {
				member.$pdfFrame.scrollTop(pdf.scrollTopValue);
			}
			
			pdf.changePaperPageNumber(pageNum);
			pdf.imageReload(pageNum, 'page', member.arrPageShowList);

			if (localSynap.properties.entireWithPartialConv == true) {
				pdf.imageReload(pageNum, 'thumb', member.arrThumbShowList);
			}
			pdf.moveThumbPage(pageNum);
			localSynap.onMovePage && localSynap.onMovePage();
			// 텍스트를 붙이기
			member.arrangePageTextAsync();
			//console.log("movePage() finish", pageNum);
		},
		
		// pageIdx : 페이지 인덱스 (0base)
		moveSlide: function (pageIdx) {
			pdf.movePage(pageIdx + 1);
		},
		
		
		// 썸네일을 해당 페이지로 이동한다.
		moveThumbPage: function (pageNum) {
			if (localSynap.hasIndexFrame()){
				var contPos = $("#thumb-area0").parent().position();
				var prevPos = $("#thumb-area" + (pageNum - 1)).parent().position();
				
				var idxScrollTopValue = parseInt( prevPos.top) - contPos.top;
				if( !($('#thumbnail').scrollTop() <= idxScrollTopValue 
					  && idxScrollTopValue + $('.imgBox').height() <= $('#thumbnail').scrollTop() + $('#thumbnail').height()) ) {
					member.$idxFrame.parent().scrollTop(idxScrollTopValue);
				}
				pdf.thumbImageEmphasized(pageNum);
			}
		},
		
		// 현재 페이지에 해당하는 썸네일 이미지에 테두리를 친다.
		thumbImageEmphasized: function (pageNum) {
			$(".imgBox.thumbnailSel").removeClass("thumbnailSel");
			$("#thumb-area" + (pageNum - 1)).parent().addClass("thumbnailSel");
		},
		
		resizePage: function (pageIndex) {
			var $page = pdf.$pages[pageIndex];
			if ($page === undefined) {
				//console.log("resizePage() undefined.",pageIndex);
				return;
			}
			var pageElem = localSynap.objList[pageIndex];
			var ratio = RATIO_NUMBERS.getRatio();
			var width = pageElem.w * ratio;
			var height = pageElem.h * ratio;

			////console.log("resizePage() start ", pageIndex, " ", $page.length);
			if ( $page.attr('src').length != 0
				 && $page.width() == width
				 && $page.height() == height) {
				//console.log("not doing");
				return;
			}
			
			// 이미지 크기 재조정
			$page[0].style.width = width + "px";
			$page[0].style.height = height + "px";

			// div 크기 재조정

			var $pageArea = pdf.$pageAreas[pageIndex];
			$pageArea[0].style.minWidth = width + "px";
			$pageArea[0].style.height = height + "px";

			// 텍스트가 있으면, 텍스트 배율도 맞춰준다.
			var $spans = $pageArea.find('span');
			var spanLen = $spans.length
			if (spanLen <= 0) {
				//console.log("resizePage() no spans");
				return;
			}
			var searchData = localSynap.searchMeta[pageIndex];
			if(BROWSER.PC.isIE()) {
				for (var index = 0; index < spanLen; ++index) {
					obj = searchData[index];
					spanStyle = $spans[index].style;
					spanStyle.top = (obj.t * ratio) + "px";
					spanStyle.left = (obj.l * ratio) + "px";
					spanStyle.height = (obj.h * ratio) + "px";
					spanStyle.fontSize = (obj.h * ratio) + "px";
				}
			} else {
				for (var index = 0; index < spanLen; ++index) {

					obj = searchData[index];
					spanStyle = $spans[index].style;
					styleStr = "top:"+(obj.t * ratio) + "px;left:" + 
						(obj.l * ratio) + "px;width:" + 
						(obj.w * ratio) + "px;height:" + 
						(obj.h * ratio) + "px;font-size:" +
						(obj.h * ratio) + "px;position:" +
						spanStyle.position;
					if (spanStyle.color) {
						styleStr += ";color:" + spanStyle.color;
					}
					if (spanStyle.backgroundColor) {
						styleStr += ";background-color:" + spanStyle.backgroundColor;
					}
					if (spanStyle.opacity) {
						styleStr += ";opacity:" + spanStyle.opacity;
					}
					styleStr += ";";
					$spans[index].style = styleStr;
				}
			}
			//console.log("resizePage() finish");
		},
		
		skinReadyFunc: function() {
			window.onresize = member.resize;
			localSynap.pathMap = {};
			
			if (localSynap.properties.isRenderServer == true) {
				member.fileInfo = localSynap.status;
				localSynap.pageSize = localSynap.status.pageNum;
			}
			if( BROWSER.isMobile() ){
				pdf.skinReadyMobileFunc();
			}else{
				pdf.skinReadyDesktopFunc();
				setResizeHeaderTitle();
			}
			containerSizeAdjust();
			setFullScreenClick();
			localSynap.onLoadedBody && localSynap.onLoadedBody();
		},

		skinReadyDesktopFunc: function() {
			slideKeyboardControl();

			if (localSynap.isAllowCopy && !localSynap.isAllowCopy()) {
				localSynap.killCopyHtml();
			}
			member.$pdfFrame = $('#contents');
			member.$idxFrame = $('#indexWrap');
			member.$idxFrame.css('margin-top', '30px');
			
			if ($('.leftPanel').length!=0){
				if (localSynap.isSingleLayout()){
					$('.leftPanel').hide();
					member.$idxFrame = undefined;
					localSynap.leftPanelShow = false;
					$('#leftPanel_hidden').css('display', 'none');
				}
				else{
					localSynap.leftPanelShow = true;
				}
			}
			
			// 헤더 아이콘 설정
			var icon = $('#iconImage');
			if (localSynap.properties.isRenderServer && localSynap.isImageMode()) {
				var fileExt = member.fileInfo.format.toLowerCase();
				// 서버에서 처리하는 루틴
				if ( fileExt == "pptx" || fileExt == "ppt") {
					icon.attr('src' , "image/header/icon_format_PPT.png");
				} else if ( fileExt == "docx" || fileExt == "doc" ) {
					icon.attr('src' , "image/header/icon_format_DOC.png");
				} else if ( fileExt == "pdf" ) {
					icon.attr('src' , "image/header/icon_format_PDF.png");
				} else if ( fileExt == "hwp" || fileExt == "hml" || fileExt == "hwp2k" || fileExt == "hwp3" ) {
					icon.attr('src' , "image/header/icon_format_HWP.png");
				} else if ( fileExt == "txt" ) {
					icon.attr('src' , "image/header/icon_format_TXT.png");
				} else if ( fileExt == "gif" || fileExt == "png" || fileExt == "jpg" || fileExt == "tiff" || fileExt == "jpeg" ) {
					icon.attr('src' , "image/header/icon_format_IMG.png");
				}
				// 초기 한정된 페이지만 정보를 받아온다.
				// 텍스트는 모두 동일하다고 생각한다.
				if (fileExt == "txt") {
					member.getImageSize(localSynap.jobId, 0);
					obj = localSynap.objList[0];
					for (var i = 0; i < localSynap.pageSize; ++i) {
						localSynap.objList[i].w = obj.w;
						localSynap.objList[i].h = obj.h;
						localSynap.objList[i].path = obj.path;
						localSynap.objList[i].index = obj.index;
					}
				} else {
					member.parseImgData(localSynap.jobId, 0);
				}
			} else {
				// 단일제품 루틴
				var fileExt = localSynap.properties.fileType || member.fileInfo.format.toLowerCase();
				if ( fileExt == "pdf" ) {
					icon.attr('src' , "image/header/icon_format_PDF.png");
					member.parsePdfXML(localSynap.properties.xmlObj);
				}
				else{
					if ( fileExt == "pptx" || fileExt == "ppt") {
						icon.attr('src' , "image/header/icon_format_PPT.png");
					} else if ( fileExt == "docx" || fileExt == "doc" ) {
						icon.attr('src' , "image/header/icon_format_DOC.png");
					} else if ( fileExt == "hwp" || fileExt == "hml" || fileExt == "hwp2k" ) {
						icon.attr('src' , "image/header/icon_format_HWP.png");
					}
					
					pdf.parseImgXml(localSynap.properties.xmlObj);
				}
			}
			
			if (localSynap.properties.isRenderServer === true) {
				setDownloadButton(localSynap.downloadUrl);
			} else {
				setDownloadButton(localSynap.properties.xmlObj);
			}
			setPrintButton();
			// ratio를 구하기위해 #contentWrap width가 결정되어 있어야 한다.
			member.$pdfFrame.css('left', localSynap.leftPanelShow ? member.getThumbnailWidth() : 0);
			member.initEvent();

			localSynap.createContentDivs();
			localSynap.createThumbDiv();
			
			var loadingCnt = localSynap.pageSize;
			if(member.isDynamicLoading() && localSynap.pageSize>pdf.INIT_MAKE_LIMIT){
				loadingCnt = pdf.INIT_MAKE_LIMIT;
			}
			
			RATIO_NUMBERS.setRatioIndex(1);
			
			if (!localSynap.isSingleLayout()){
				localSynap.loadThumbImage(0, loadingCnt);
			}

			pdf.changePaperPageNumber(localSynap.getCurrentPage());
			member.arrangePageTextAsync();
		},

		skinReadyMobileFunc: function() {
			initMobile();
			member.$pdfFrame = $('#contents');

			if (localSynap.isImageMode()) {
				member.parseImgData(localSynap.jobId, 0);
			} else {
				member.parsePdfXML(localSynap.properties.xmlObj);
			}
			
			for ( var i = 1; i <= localSynap.pageSize; ++i ) {
				$('.paging>select').append('<option>' + i + '</option>');	
			}
			
			$('.paging').append(' / ' + localSynap.pageSize);
			$('.paging>select').change(function(){
				$("select option:selected").each(function(){
					pdf.movePage($(this).index()+1);
				});
			});
			
			member.initEvent();
			
			// div를 먼저 생성한다.
			
			localSynap.createContentDivs();
			localSynap.createThumbDiv();
			
			// 갤노트3 기본브라우저에서는 wrap 높이를 지정해주어야 헤더가 스크롤 시 빨려올라가지 않는다.
			if ( BROWSER.MOBILE.isGalaxyNote3() ) {
				$('#wrap').css('height', $(window).height() - $('#header').height());
			}
			var loadingCnt = localSynap.pageSize;
			if(member.isDynamicLoading() && localSynap.pageSize>pdf.INIT_MAKE_LIMIT && !BROWSER.VERSION.isAndroidICS() ){
				loadingCnt = pdf.INIT_MAKE_LIMIT;
			} else {
				// ICS는 동적로딩을 하지 않는다.
				pdf.INIT_MAKE_LIMIT = localSynap.pageSize;
			}
			
			
			
			// 렌더링 완료 시점에서 초기화하는 명령들
			RATIO_NUMBERS.setRatioIndex(1);
		}
		,
		showlist: function (){
			//console.log(member.arrPageShowList);
			//console.log(member.arrThumbShowList);
		}
	}
	return pdf;
})());

localSynap.getPageSize = function() {
	return localSynap.pageSize;
}

localSynap.getCurrentPage = function() {
	if (localSynap.curPage < 1) {
		return 1;
	} else {
		return localSynap.curPage;
	}
}

localSynap.killCopyHtml = function() {
	// common
	var contents = document.getElementById('contents');
	var thumbnail = document.getElementById('thumbnail');
	stopBrowserEvent(contents);
	stopBrowserEvent(thumbnail);
	thumbnail.onload=function(){
		stopBrowserEvent(contents);
		stopBrowserEvent(thumbnail);
	};
}


var RATIO_NUMBERS = {
	curIndex : 0,
	// 배율에 대한 비례값 (변환서버용)
	standardRatio: 1,
	numbers : [1, 1.5, 2],
	increaseRatio : function(){
		if (0 <= this.curIndex && this.curIndex < (this.numbers.length-1)) {
			++this.curIndex;
		}
		return this.numbers[this.curIndex] * this.standardRatio;
	},
	reduceRatio : function() {
		if (0 < this.curIndex && this.curIndex <= (this.numbers.length-1)) {
			--this.curIndex;
		}
		return this.numbers[this.curIndex] * this.standardRatio;
	},
	setRatioIndex : function(_ratio) {
		this.curIndex = this.numbers.length - 1;
		if (_ratio > this.numbers[this.numbers.length - 1]) {
			localSynap.ratio = this.numbers[this.numbers.length - 1];
			return;
		}
		for (var i=0; i<this.numbers.length; i++) {
			if (this.numbers[i] == _ratio) {
				this.curIndex = i;
				break;
			}
		}
	},
	setStandardRatio : function (value) {
		this.standardRatio = value;
	},
	getRatio : function () {
		return this.numbers[this.curIndex] * this.standardRatio;
	},
	getValueOfZoom : function (index) {
		return this.numbers[index] * this.standardRatio;
	}
};
function thumb_retry(idx)
{
	if (localSynap.properties.entireWithPartialConv == true) {
		console.log("thumb_retry");
		localSynap.setImage("thumb", idx, function (pageIndex, srcPath) {
			$('#thumb'+pageIndex).attr('src' , localSynap.getResultDir() + localSynap.objList[pageIndex].path).attr('onclick', '').attr('alt', 'thumb'+(pageIndex*1+1));
		});
	} else {
		$('#thumb'+idx).attr('src' , localSynap.getImagePath(idx)).attr('onclick', '').attr('alt', 'thumb'+(idx*1+1));
	}
}
function image_thum_error(obj)
{
	$(obj).attr('onclick', 'thumb_retry('+obj.name+')');
}
function image_retry(idx)
{
	if (localSynap.properties.entireWithPartialConv == true) {
		console.log("page_retry");
		localSynap.setImage("page", idx, function (pageIndex, srcPath) {
			localSynap.$pages[pageIndex].attr('src' , localSynap.getResultDir() + localSynap.objList[pageIndex].path).attr('onclick', '').attr('alt', 'page'+(pageIndex*1+1));
			localSynap.resizePage(pageIndex);
		});
	} else {
		localSynap.$pages[idx].attr('src' , localSynap.getImagePath(idx)).attr('onclick', '').attr('alt', 'page'+(idx*1+1));
		localSynap.resizePage(idx);
	}
}


function image_error(obj)
{
	$(obj).attr('onclick', 'image_retry('+obj.name+')');
}



function append_text_async(xmlObj) {
	var index = xmlObj.index;
	var $page = localSynap.$pages[index];
	if (localSynap.properties.usePdfText === false) { return; }	
	if (localSynap.$pageAreas[index].find("span").length == 0 && localSynap.searchMeta[index] ) {
		
		//console.log('append_text_async() ', xmlObj.index);
		startTime = new Date();
		var $pageArea = localSynap.$pageAreas[index];
		// spanString을 만드는 도중, 텍스트출력큐에 내용이 들어오면
		// 다른페이지를 출력해야 할 때, 바로 종료한다.
		var lastQueueIndex = pdfSearch.textOutputQueue.length-1;
		if (pdfSearch.textOutputQueue.length > 0 
			&& pdfSearch.textOutputQueue[lastQueueIndex] != index
			&& pdfSearch.textOutputQueue[lastQueueIndex-1] != index) {
			//console.log("append span break!!!!!");
			return false;
		}
		if (BROWSER.PC.isIE() && BROWSER.VERSION.IE() <= 9) {
			localSynap.appendSpanTags(index);
		} else {
			$pageArea.append(localSynap.getText(index));
		}

		endTime = new Date();
		elapse = endTime.getTime() - startTime.getTime();
		//console.log('append_text_async() finish', xmlObj.index," ", elapse );
	}

	if (localSynap.properties.webAccessibility) {
		localSynap.prepareFullText(index);
		$page.attr('alt', localSynap.fullText[index]);
	}
}

var onLoadImg = function(index){
	if( BROWSER.isMobile() ){
		// 모바일은 화면맞춤을 위해 resize하지 않는다.
	}else{
		if (++localSynap.loadedImgCount === localSynap.pageSize) {
			localSynap.resizePage(index);
		}
	}
}

var cleanSpans = function (pageIndex) {
	//console.log('cleanSpan() start', pageIndex);
	startTime = new Date();
	len = localSynap.pageSize;
	for (var targetIdx = 0; targetIdx < len; ++targetIdx) {
		// find가 너무 자주 수행되므로, 캐싱한다면 성능향상이 기대된다.
		$spans = localSynap.$pageAreas[targetIdx].find('span');
		if( $spans.length === 0) {
			continue;
		}
		//if (targetIdx < pageIndex-1 || targetIdx > pageIndex+2) {
		if (targetIdx != pageIndex) {
			//console.log('cleaning ', targetIdx);
			// 성능개선요소
			//pa = document.getElementById('page-area'+targetIdx);
			//imgTag = pa.firstChild;
			//pa.innerHTML = imgTag.outerHTML;
			$spans.remove();
		}
	}
	endTime = new Date();
	//console.log('cleanSpan() finish ', pageIndex, " ", endTime.getTime() - startTime.getTime());
}


// SKIN READY FUNC
$(document).ready(function() {
	if (typeof localSynap.skinReadyFunc == "function") {
		localSynap.skinReadyFunc();
	}
});

function loadSpinner(pageClassName, callback){
	$('.loading_spinner').remove();
	
	var opts = {
		lines: 11 // The number of lines to draw
		, length: 0 // The length of each line
		, width: 12 // The line thickness
		, radius: 30 // The radius of the inner circle
		, scale: 1.25 // Scales overall size of the spinner
		, corners: 0.5 // Corner roundness (0..1)
		, color: '#435c85' // #rgb or #rrggbb or array of colors
		, opacity: 0.25 // Opacity of the lines
		, rotate: 0 // The rotation offset
		, direction: 1 // 1: clockwise, -1: counterclockwise
		, speed: 1 // Rounds per second
		, trail: 65 // Afterglow percentage
		, fps: 20 // Frames per second when using setTimeout() as a fallback for CSS
		, zIndex: 2e9 // The z-index (defaults to 2000000000)
		, className: 'spinner' // The CSS class to assign to the spinner
		, top: '50%' // Top position relative to parent
		, left: '50%' // Left position relative to parent
		, shadow: false // Whether to render a shadow
		, hwaccel: false // Whether to use hardware acceleration
		, position: 'absolute' // Element positioning
	}
	var target = document.createElement('div'); target.setAttribute('id', 'div_page'); target.setAttribute('class', 'inner loading_spinner ' + pageClassName);
	document.getElementById('container').appendChild(target);
	var spinner = new Spinner(opts).spin(target);
	callback && callback();
}

function removeSpinner(){
	setTimeout(function(){
		$('.loading_spinner').remove();
	}, 50);
}



