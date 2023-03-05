/*jslint devel: true, browser: true*/
/*global define, $, localSynap, statusTimer, BROWSER*/
var api = (function () {
	"use strict";
	var member, api = {
		loadCommon : function () {
			var head = document.getElementsByTagName('head')[0],
				common = document.createElement('script');
			common.type = 'text/javascript';
			common.src = 'common_skin.js';
			/*common.onload = function(){
				var common_skin = document.createElement('script');
				common_skin.type = 'text/javascript';
				common_skin.src = 'common_skin.js';
				head.appendChild(common_skin);
			}*/
			head.appendChild(common);
		},
		getSkinType : function (fileExt) {
			var skinType = "";
			if (fileExt === "xls" || fileExt === "xlsx" || fileExt === "csv") {
				skinType = "cell";
			} else if (fileExt === "ppt" || fileExt === "pptx") {
				skinType = "slide";
			} else if (fileExt === "doc" || fileExt === "docx" || fileExt === "hwp2k" || fileExt === "hwp97" || fileExt === "txt") {
				skinType = "word";
			} else if (fileExt === "jpeg" || fileExt === "jpg" || fileExt === "png" || fileExt === "tiff" || fileExt === "gif") {
				skinType = "image";
			} else if (fileExt === "pdf") {
				skinType = "pdf";
			} else {
				skinType = "word";
			}
			return skinType;
		},
		
		parseStatus : function (id, contextPath) {
			localSynap.jobId = id;
			localSynap.properties.contextPath = contextPath;
			var fileExt = "",
				escCnt = 0,
				foo = function () {
					if (!(typeof localSynap.status === "undefined" || (localSynap.status.imgDone === false && localSynap.status.htmlDone === false))) {
						clearInterval(statusTimer);
						if (localSynap.status.imgErrCode > 0) {
							console.log('image converting Error : Code ' + localSynap.status.imgErrCode);
							location.href = contextPath + '/exception';
						} else if (localSynap.status.htmlErrCode > 0) {
							console.log('HTML converting Error : Code ' + localSynap.status.htmlErrCode);
							location.href = contextPath + '/exception';
						} else if (localSynap.status.pdfErrCode > 0) {
							console.log('PDF converting Error : Code ' + localSynap.status.pdfErrCode);
						//location.href = contextPath + '/exception';
						}
						return;
					}
				
					$.ajax({
						type: 'GET',
						url: contextPath + '/status/' + id,
						async: false,
						dataType: 'json',
						success: function (obj) {
							var statusObj = {
								id: -1,
								convertType: 0,
								format: null,
								fileName: "",
								refererUrl: "",
								downloadUrl: "",
								pageNum: 0,
								imgDone: false,
								htmlDone: false,
								pdfDone: false
							};
							if (obj.format === null) {
								localSynap.properties.fileType = 'docx';
							} else {
								localSynap.properties.fileType = obj.format.toLowerCase();
							}

							localSynap.properties.title = obj.fileName;
							//localSynap.properties.fileName = obj.fileName;
							localSynap.downloadUrl = obj.downloadUrl;
							localSynap.status = obj;
							if (localSynap.status.convertType === 0) {
								api.getXmlFromServer(id, contextPath);
							} else {
								api.getImage(id, contextPath);
							}
						}
					});
				},
				statusTimer = setInterval(foo, 500);
		},
		
		getImage : function (key, contextPath) {
			localSynap.jobId = key;
			localSynap.properties.contextPath = contextPath;
			var fileExt = localSynap.status.format.toLowerCase(),
				skinType = api.getSkinType(fileExt),
				htmlUrl,
				head,
				script;
			
			if (skinType) {
				htmlUrl = skinType;
				if (skinType !== "cell") {
					htmlUrl = "pdf";
				}
				if (BROWSER.isMobile()) {
					htmlUrl = htmlUrl + '_skin_mobile.xhtml';
				} else {
					htmlUrl = htmlUrl + '_skin.html';
				}
				
				// text검색모듈
				// TODO: 이 위치에 있어도, 본문js 보다 먼저 import되리란 보장이 없다. document.html에 넣어야 할 수도 있다.
				head = document.getElementsByTagName('head')[0];
				script = document.createElement('script');
				script.type = 'text/javascript';
				script.src = 'pdf_search.js';
				head.appendChild(script);
				
				$.ajax({
					type: 'GET',
					url: htmlUrl,
					dataType: 'text',
					success: function (html) {
						$('body').append(html);
						$('#headTitle').append('<span>' + localSynap.properties.title + '</span>');
						/*$.ajax({
							type: 'GET',
							url: skinType + '_skin.js',
							dataType: 'text',
							success:function(script){
								eval.call( window, script );
							},
							error:function(err){
								eval( err.responseText );
							}
						});*/
						// For debugging, not checked in all browser.
						var head = document.getElementsByTagName('head')[0],
							script = document.createElement('script');
						script.type = 'text/javascript';
						if (skinType === "cell") {
							script.src = 'cell_skin.js';
						} else {
							script.src = 'pdf_skin.js';
						}
						head.appendChild(script);
					},
					error: function (err) {
						console.log(err.responseText);
					}
				});
			}
		},
		getXmlFromServer : function (id, contextDir) {
			localSynap.properties.xmlUrl = contextDir + 'xml/' + id;
			localSynap.jobId = id;
			$.ajax({
				type: 'GET',
				async: false,
				url: localSynap.properties.xmlUrl,
				dataType: 'xml',
				success: function (xmlDoc) {
					api.loadDoc({xml: xmlDoc});
				},
				error: function (error) {
					console.log(error.responseText);
				}
			});
		},
		
		getXml : function (fileName, rsDir) {
			localSynap.properties.resultDir = rsDir[rsDir.length - 1] === '/' ? rsDir : rsDir + '/';
			localSynap.properties.fileName = fileName;
			// TODO(MINO) : 이미지일경우 xml 파일의 경로는 아래와 다르다.
			if (localSynap.properties.entireWithPartialConv == true) {			
				localSynap.properties.xmlUrl = localSynap.properties.resultDir + localSynap.properties.fileName + '.xml';
			} else {
				localSynap.properties.xmlUrl = localSynap.properties.resultDir + localSynap.properties.fileName + '.xml';
			}
			//localSynap.properties.xmlUrl = localSynap.properties.resultDir + localSynap.properties.fileName + '/' + localSynap.properties.fileName + '.xml';
			$.ajax({
				type: 'GET',
				async: false,
				url: encodeURI(localSynap.properties.xmlUrl),
				dataType: 'xml',
				success: function (xmlDoc) {
					api.loadDoc({xml: xmlDoc});
				},
				error: function (error) {
					console.log(error.responseText);
				}
			});
		},
		setCurrentSheet : function (sheetNo) {
			localSynap.curPage = sheetNo;
		},
		loadDoc : function (obj) {
			localSynap.properties.xmlObj = $(obj.xml);
			var xmlInfo = {},
				skinType = "",
				strHref,
				xmlFileName,
				htmlUrl,
				head,
				script;
			
			// 이미지변환기 or HTML변환기 선택
			xmlInfo.convType = $(obj.xml).find('convert_type').text();
			localSynap.properties.fileType = $(obj.xml).find('file_type').text();
			 
			if (xmlInfo.convType === "image") {
				// 이미지변환기도 pdf_skin을 사용한다.
				skinType = "pdf";
			} else { // if (xmlInfo.convType == "html")
				skinType = api.getSkinType(localSynap.properties.fileType);
			}
			if ($(obj.xml).find('use_single_page').text() === 'true') {
				localSynap.properties.layout = 'single';
			} else {
				localSynap.properties.layout = 'withpage';
			}
			if (BROWSER.PC.isIE() && BROWSER.VERSION.IE() <= 9 && skinType === "slide" && ieCompatible === 8) { // Todo : 9이상 브라우저를 체크를 해야하는지?
				strHref = location.href.replace(/doc.html/ , 'doc_7.html');
				location.href = strHref;
			} else if (BROWSER.PC.isIE() && skinType !== "slide" && ieCompatible === 7) {
				strHref = location.href.replace(/doc_7.html/, 'doc.html');
				location.href = strHref;
			}
			
			// 렌더서버가 있을 때는 status의 파일명을 사용한다. 
			if (localSynap.properties.isRenderServer === true) {
				localSynap.properties.title = localSynap.status.fileName;
				localSynap.properties.fileName = $(obj.xml).find('file_name').text();
			} else {
				// 없을 때에는 파일명>문서속성제목 순으로 한다.
				xmlInfo.title = $(obj.xml).find('doc_title').text();
				if (xmlInfo.title.length === 0) {
					xmlInfo.title = $(obj.xml).find('file_name').text();
				}
				localSynap.properties.title = xmlInfo.title;
			}
			
			if (localSynap.properties.isRenderServer === true) {
				if (localSynap.isImageMode()) {
					xmlFileName = localSynap.jobId;
				} else {
					xmlFileName = $(obj.xml).find('file_name').text();
				}

				localSynap.properties.resultDir = localSynap.properties.contextPath + 'result/' + xmlFileName + '/';
			}
			
			if (skinType === "cell" || skinType === "csv") {
				localSynap.pageSize = $(obj.xml).find('sheet_cnt').text();
			} else if (skinType === "slide") {
				localSynap.pageSize = $(obj.xml).find('slide_cnt').text();
			} else if (skinType === "pdf") {
				if (localSynap.properties.entireWithPartialConv == true) {
					localSynap.pageSize = $(obj.xml).find('pdf_cnt').text();
				} else {
					var $endPage = $(obj.xml).find('endPage');
					var $startPage = $(obj.xml).find('startPage');
					if ($endPage.length == 0 && $startPage.length == 0) {
						localSynap.pageSize = $(obj.xml).find('pdf_cnt').text();
					} else {
						localSynap.pageSize = 1 + $endPage - $startPage;
					}
				}
			} else {
				localSynap.pageSize = $(obj.xml).find('image_cnt').text();
			}
			
			if (!localSynap.properties.debug) {
				$.ajaxSetup({cache: true});
			}
			if (skinType) {
				htmlUrl = skinType;
				
				if (BROWSER.isMobile()) {
					htmlUrl = htmlUrl + '_skin_mobile.xhtml';
				} else {
					htmlUrl = htmlUrl + '_skin.html';
				}
				
				// pdf 검색모듈 로딩
				if (skinType === 'pdf') {
					head = document.getElementsByTagName('head')[0];
					script = document.createElement('script');
					script.type = 'text/javascript';
					script.src = 'pdf_search.js';
					head.appendChild(script);
				}
				$.ajax({
					type: 'GET',
					url: htmlUrl,
					dataType: 'text',
					success: function (html) {
						$('body').append(html);
						$('#headTitle').append(localSynap.properties.title);
						/*$.ajax({
							type: 'GET',
							url: skinType + '_skin.js',
							dataType: 'text',
							success:function(script){
								eval.call( window, script );
							},
							error:function(err){
								eval( err.responseText );
							}
						});*/
						// For debugging, not checked in all browser.
						var head = document.getElementsByTagName('head')[0],
							script = document.createElement('script');
						script.type = 'text/javascript';
						script.src = skinType + '_skin.js';
						head.appendChild(script);
					},
					error: function (err) {
						console.log(err.responseText);
					}

				});
			}
		},
		// TODO : 한글 인코딩 지원
		getParameter : function (param) {
			var returnValue,
				url = location.href,
				parameters = (url.slice(url.indexOf('?') + 1, url.length)).split('&'),
				i,
				varName;
			for (i = 0; i < parameters.length; i = i + 1) {
				varName = parameters[i].split('=')[0];
				if (varName.toUpperCase() === param.toUpperCase()) {
					returnValue = parameters[i].split('=')[1];
					return decodeURIComponent(returnValue);
				}
			}
			return undefined;
		},
		referrer : function (param) {
		},
		errorMsg : function () {
			// 잘못된 ***
			alert('잘못된 파일 이름입니다');
		},
		printDocFromServer : function(id, contextDir){
			var printUrl = contextDir + 'pdf/' + id;
			localSynap.jobId = id;
			if(localSynap.status.pdfErrCode > 0){
					alert("인쇄 작업에 실패했습니다.");
					return;
			}
			$.ajax({
					type: 'GET',
					async: false,
					url: printUrl,
					success: function(xmlDoc){
							// FIXME : Duplicate Download Call
							window.location = printUrl;
					},
					error: function(error){
							alert("인쇄 작업에 실패했습니다.");
					}
			});
		}
	};
	return api;
}());

function sleep(num) {	//[1/1000초]
	"use strict";
	var now = new Date(),
		stop = now.getTime() + num;
	while (true) {
		now = new Date();
		if (now.getTime() > stop) {
			return;
		}
	}
}
