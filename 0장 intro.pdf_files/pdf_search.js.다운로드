/*
PDF 검색모듈
텍스트정보XML을 읽고, 각 페이지 별 span태그 목록을 반환한다.

예제 코드
var aaa = new pdfSearch("test.xml");
$(target).append(aaa.getData(2));
*/
console.log("pdfSearch on");

var pdfSearch = (function() {
	var Search = {
		textObj : {
			l: 0,
			t: 0,
			w: 0,
			h: 0,
			text: "",
			link: null
		},
		cir: undefined,
		findList: {},
		findPageIdx: 0,
		findTextIdx: undefined,
		nextStartIdx: undefined,
		loadXmlQueue: [],
		textOutputQueue: [],
		ret: false,
		storage: {},
		setPageTextIndex: function (pageNum, text) {
			key = localSynap.getFileName() + "_" + pageNum;
			if (typeof(Storage) !== "undefined") {
				// localStorage
				localStorage.setItem(key, text);
			} else {
				// no localStorage
				pdfSearch.storage[key] = text;
			}
		},
		getPageTextFromIndex: function (pageNum) {
			key = localSynap.getFileName() + "_" + pageNum;
			if (typeof(Storage) !== "undefined") {
				return localStorage.getItem(key);
			} else {
				return pdfSearch.storage[key];
			}
		},
		hasPageTextIndex: function (pageNum) {
			key = localSynap.getFileName() + "_" + pageNum;
			if (typeof(Storage) !== "undefined") {
				val = localStorage.getItem(key);
				if (val == null || val == undefined) {
					return false;
				}
				// IE8/9에서는 item값이 null 이어도 빈문자열을 삽입하기 때문에, length체크를 해야한다.
				// 빈 페이지의 경우, 인덱스되지 않은 것으로 판단할 수 있지만, 구버전IE에서는 판단할 수 없으므로 안고간다.
				return val.length > 0;
			} else {
				return pdfSearch.storage[key] !== undefined;
			}
		},
		getTextOffsetFromPage: function (text, pageIdx, textOffset) {
			pgText = pdfSearch.getPageTextFromIndex(1+pageIdx);
			return pgText.indexOf(text, textOffset);
		},
		getNextOffsetFromPage: function (text, pageIdx, textOffset) {
			pgText = pdfSearch.getPageTextFromIndex(1+pageIdx);
			return pgText.indexOf(text, 1+textOffset);
		},
		getPrevOffsetFromPage: function (text, pageIdx, textOffset) {
			pgText = pdfSearch.getPageTextFromIndex(1+pageIdx);
			if (BROWSER.VERSION.IE() == 11) {
				substr = pgText.substring(0, textOffset-1);
				return substr.lastIndexOf(text);
			} else {
				return pgText.lastIndexOf(text, textOffset-1);
			}
		},
		findallOffsetsFromPage: function (text, pageIdx) {
			pdfSearch.loadPageText(1+pageIdx);
			pgText = pdfSearch.getPageTextFromIndex(1+pageIdx);
			list = []
			offset = 0;
			while(1) {
				idx = pgText.indexOf(text, offset);
				if (idx < 0) {
					break;
				}
				list.push(idx);
				offset = idx + 1;
			}
			return list;
		},
		updateHighlightQueue: function (index, text) {
			// 현재페이지 부터 다음 검색결과가 나올 때까지 찾는다.
			if (pdfSearch.findList[text][index] !== undefined) {
				offsets = pdfSearch.findList[text][index];
			} else {
				offsets = pdfSearch.findallOffsetsFromPage(text, index);
				// 페이지의 FindList셋을 채운다. 
				pdfSearch.findList[text][index] = offsets;
				// 아래 코드가 필요할 수 있음.
				/*
				if (offsets.length == 0) {
					pdfSearch.findList[text][index] = null;
				}
				*/
			}
			find_list = pdfSearch.findList[text];
			// 그 외의 범위는 queue에 담아, 나중에 출력될 때 처리하고 업데이트 한다.
			pushData = {"text": text, list:find_list};
			localSynap.highlightQueue = pushData;
			return offsets;
		},
		savePageText: function (pageIndex) {
			var filePath = localSynap.getResultDir();
			if (localSynap.properties.entireWithPartialConv == true) {
				filePath += localSynap.properties.resultPath;
			}
			filePath += localSynap.properties.fileName + ".files/" + localSynap.properties.fileName + "_" + (1+pageIndex) + ".xml";
			if (window.XMLHttpRequest) {
				xhttp = new XMLHttpRequest();
			} else {
				xhttp = new ActiveXObject("Microsoft.XMLHTTP");
			}

			xhttp.overrideMimeType && xhttp.overrideMimeType('text/xml');
			xhttp.open("GET", filePath, false);
			xhttp.send(null);

			xmlDoc = xhttp.responseXML;
		strBuf = "";
		$lists = $(xmlDoc).find("page text");
		len = $lists.length;
			if (BROWSER.VERSION.IE() <= 9) {
				for ( var idx = 0; idx < len; ++idx) {
					if ($lists[idx].text.length >= 1) {
						strBuf += $lists[idx].text;
					} else {
						strBuf += " ";
					}
				}
			} else {
				for ( var idx = 0; idx < len; ++idx) {
					textChar = $($lists[idx]).text(); 
					if (textChar.length >= 1) {
						strBuf += textChar;
					} else {
						strBuf += " ";
					}
				}		
			}	
		pageText = strBuf.toLowerCase();

			pdfSearch.setPageTextIndex(pageIndex+1, pageText);

		},
		loadPageText: function (pageNum) {
			////console.log("loadPageText()", pageNum);
		  	// 인덱스가 없으면 XML을 읽어온다.
			if (pdfSearch.hasPageTextIndex(pageNum)) {
			} else {
				pdfSearch.savePageText(pageNum-1);
			}
			return;
		},
		findTextInPage: function (index, text, isPrev) {
			index = parseInt(index);
		  	var pageNum = index+1;
			var pgIndex = localSynap.getCurrentPage()-1;
			
			accessToSearch = function () {
				isFound = false;
				//console.log("on search");
				offsets = pdfSearch.updateHighlightQueue(index, text);	

				// 현재 페이지에 해당 텍스트가 있으면
				if (offsets.length > 0) {
					// 현재 페이지 및 텍스트인덱스로 오프셋을 찾아낸다.
					if (isPrev) {
						textOffset = pdfSearch.getPrevOffsetFromPage(text, index, pdfSearch.findTextIdx);
					} else {
						if (pdfSearch.nextStartIdx == undefined) {
							textOffset = pdfSearch.getTextOffsetFromPage(text, index, pdfSearch.findTextIdx);
						} else {
							textOffset = pdfSearch.getNextOffsetFromPage(text, index, pdfSearch.findTextIdx);
						}
					}
					// 현재 페이지에서 텍스트 오프셋이 더이상 존재하지 않으면, 다음 페이지로 넘어가야 한다.
					if (textOffset >= 0) {
						// 현재페이지 기억
						pdfSearch.findPageIdx = index;
						// 현재텍스트 인덱스 기억
						pdfSearch.findTextIdx = textOffset;
						pdfSearch.nextStartIdx = textOffset+1;
						isFound = true;
						////console.log(text + " has offset " + textOffset + " on pageIndex " + index);
						removeSpinner();
					} else {
						if (isPrev) {
							pdfSearch.findTextIdx = 9999999;
							pdfSearch.findPageIdx -= 1;
						} else {
							pdfSearch.findTextIdx = 0;
							pdfSearch.findPageIdx += 1;
						}
					}

				}
				if (isFound == false) {
					return;
				}
				// 페이지이동
				if (localSynap.getCurrentPage() === parseInt(pdfSearch.findPageIdx)+1 ) {
				} else {
					localSynap.movePage(parseInt(pdfSearch.findPageIdx)+1);
				}
				// 출력되어있는 span을 업데이트 한다.
				for (index in pdfSearch.findList[text]) {
					// 앞뒤 페이지들은 span이 없을 것이라 생각하고 건너뛴다.
					if (index <= pgIndex - 3 || pgIndex + 3 <= index) {
						continue;
					}
					for (txtIdx in pdfSearch.findList[text][index]) {
						localSynap.setHighlight(index, pdfSearch.findList[text][index][txtIdx], text.length);
					}
				}

				return isFound;

			};

			pdfSearch.loadPageText(pageNum);
			return accessToSearch();
		},
		makeAttrObj: function (data) {
			var obj = {};
			if (typeof data === "undefined") { return obj; }
			obj.t = data.getAttribute('t');
			obj.l = data.getAttribute('l');
			obj.w = data.getAttribute('w');
			obj.h = data.getAttribute('h');

			if(BROWSER.PC.isIE()) {
				if(typeof data.nodeTypedValue!=="undefined") {
					obj.text = data.nodeTypedValue;
				} else {
					if(typeof data.textContent!=="undefined") {
						obj.text = data.textContent;
					}
				}
			} else {
				obj.text = data.textContent; // IE의 경우는 IE9에서만 지원된다.
			}
			return obj;
		},
		callAjaxXmlData: function (xmlPath, pageIndex, pushTextFunc) {
			$.ajax({
				type: "GET",
				url: xmlPath,
				//async: false,
				dataType: (BROWSER.PC.isIE()) ? "text" : "xml",
				error: function(data){
					//console.log('Error occurred loading Text XML : ', xmlPath, data);
				},
				success:function(data) {

					if (typeof data === "string") {
						xml = new ActiveXObject("Microsoft.XMLDOM");
						xml.async = false;
						xml.loadXML(data);
					} else {
						xml = data;
					}

					pushTextFunc($(xml).find("page"), pageIndex);

					// 저장해둔다
					strBuf = "";
					$lists = $(xml).find("page text");
					len = $lists.length;
					if (BROWSER.VERSION.IE() <= 9) {
						for ( var idx = 0; idx < len; ++idx) {
							if ($lists[idx].text.length >= 1) {
								strBuf += $lists[idx].text;
							} else {
								strBuf += " ";
							}
						}
					} else {
						for (var idx = 0; idx < len; ++idx) {
							textChar = $($lists[idx]).text();
							if (textChar.length >= 1) {
								strBuf += textChar;
							} else {
								strBuf += " ";
							}
						}
					}
					pageText = strBuf.toLowerCase();

					pdfSearch.setPageTextIndex(pageIndex+1, pageText);

				}
			});
		},
		makeXmlDataObj: function (xmlObj, textList, startX, startY, endX, endY, isRelative) {
			var page = $(xmlObj).attr('page');
			var target = $(xmlObj).attr('target');
			if ( (typeof page !== "undefined") || (typeof target !== "undefined") ) {
				$(xmlObj).children().each(function (lidx, lxmlObj) {
					obj = Search.makeAttrObj(lxmlObj);
					if(typeof page !== "undefined") {
						obj.link = parseInt(page); // 페이지 번호로 이동하는 link는 숫자형으로 기록한다.
					} else {
						obj.link = target;
					}
					// check boundary
					if(typeof isRelative==="boolean" && isRelative==true) {
						//셀의 경우 잘라진 조각 이미지를 기준으로 좌표이기 때문에 좌표 조정이 필요하다.
						obj.l = parseFloat(obj.l) + (startX-1);
						obj.t = parseFloat(obj.t) + (startY-1);
					}
					if(typeof startX!=="undefined") {
						var objLeft = parseFloat(obj.l);
						var objTop = parseFloat(obj.t);
						if(objLeft>=startX && objLeft<=endX && objTop>=startY && objTop<=endY) {
							textList.push(obj);
						}
					} else {
						textList.push(obj);
					}
				});
			} else {
				obj = Search.makeAttrObj(xmlObj);
			}
			if ( (typeof page === "undefined") && (typeof target === "undefined") ) {
				// check boundary
				if(typeof isRelative==="boolean" && isRelative==true) {
					//셀의 경우 잘라진 조각 이미지를 기준으로 좌표이기 때문에 좌표 조정이 필요하다.
					obj.l = parseFloat(obj.l) + (startX-1);
					obj.t = parseFloat(obj.t) + (startY-1);
				}
				if(typeof startX!=="undefined") {
					var objLeft = parseFloat(obj.l);
					var objTop = parseFloat(obj.t);
					if(objLeft>=startX && objLeft<=endX && objTop>=startY && objTop<=endY) {
						textList.push(obj);
					}
				} else {
					textList.push(obj);
				}
			}

			return [obj, textList];
		},
		// input : 텍스트정보가 있는 xml파일
		// rect영역내의 텍스트만 읽어들이도록 boundary정보를 받을 수 있다.
		loadTextXml: function(xmlFilePath, startX, startY, endX, endY, isRelative) {
			var textList = [];
			var xmlData = pdfSearch.callAjaxXmlData(xmlFilePath);   
			var obj = {};
			$.each($(xmlData).children(), function (idx, elem) {
				var retObj = pdfSearch.makeXmlDataObj(elem, textList);
				obj = retObj[0];
				textList = retObj[1];
			});
			var pageSizeObj = {
				width: xmlData.attr('w'),
				height: xmlData.attr('h')
			};
			// TODO: 이거 함수로 꼭 빼내야됨. 급해서 이렇게 해놓은거
			localSynap.pageInfo = pageSizeObj;
			return textList;
		},
		loadTextXmlAsync: function(xmlFilePath, pageIdx, startX, startY, endX, endY, isRelative) {
			pushTextFunc = function (xmlData, pageIndex) {
				var obj = {};
				var textList = [];
				var arrText = $(xmlData).children();
				if (arrText.length === 0) { return; }
				for (var idx = 0; idx < arrText.length; ++idx) {
					////console.log("loadTexmlXmlAsync..");
					var elem = arrText[idx];
					var retObj = pdfSearch.makeXmlDataObj(elem, textList);
					// obj = retObj[0];
					textList = retObj[1];
				}
				// TODO: 이거 함수로 꼭 빼내야됨. 급해서 이렇게 해놓은거
				localSynap.pageInfo = {
					width: xmlData.attr('w'),
					height: xmlData.attr('h')
				};
				targetObj = {
					index: pageIdx,
					content: textList
				};
				localSynap.appendText(targetObj, append_text_async);
			}

			if (localSynap.searchMeta[pageIdx] == undefined) {
				pdfSearch.callAjaxXmlData(xmlFilePath, pageIdx, pushTextFunc);
			} else {
				targetObj = {
					index: pageIdx,
					content: localSynap.searchMeta[pageIdx]
				};
				localSynap.appendText(targetObj, append_text_async);
			}
		},
		/*
		인덱스 페이지의 텍스트 정보를 담은 Span태그들을 반환한다.
		input : pageIdx 페이지 인덱스
				offsetX, offsetY : X좌표와 Y좌표를 보정해야 하는경우 좌표 수치
		output : ex. <span class="pageText" style="top:10px;left:40px">.....</span>...</span>
		*/
		getSpanString: function(pageIdx, parsedData, offsetX, offsetY) {
			if(localSynap.properties.useMobileTextSearch == false && BROWSER.isMobile())
				return;
			
			var textNodes = parsedData[pageIdx];
			if (textNodes == undefined) { return; }
			var html = '';
			var opacityZeroString;
			if (BROWSER.PC.isIE()) {
				if (BROWSER.VERSION.IE() <= 7) {
					opacityZeroString = "filter: alpha(opacity=0); ";
				} else if (BROWSER.VERSION.IE() == 8) {
					opacityZeroString = "-ms-filter:\"progid:DXImageTransform.Microsoft.Alpha(Opacity=0)\"; ";
				} else {
					opacityZeroString = "color:transparent;";
				}
			} else {
				opacityZeroString = "color:transparent;";
			}

			// 이미지변환기용 스킨은 원본이미지와 크기가 다른 경우가 있다.
			// 텍스트를 배치할 때 그 비율이 필요하며, 아래의 코드가 사용된다. 다소 손볼곳이 많으나, 구조가 안정화 된 후에 정리하자
			if (localSynap.pxToPtRatio == undefined && localSynap.isImageMode()) {
				localSynap.pxToPtRatio = parseInt(localSynap.$pageAreas[0].css('width')) / localSynap.pageInfo.width;
				RATIO_NUMBERS.setStandardRatio(localSynap.pxToPtRatio);
			}
			var adjustRatio =  RATIO_NUMBERS.getRatio();
			var counter = 0;
			var lastIndex = pdfSearch.textOutputQueue.length-1;
			$.each(textNodes, function (idx, elem) {
				// spanString을 만드는 도중, 텍스트출력큐에 내용이 들어오면
				// 다른페이지를 출력해야 할 때, 바로 종료한다.
				// 두 페이지만 텍스트출력을 허용한다.
				if (pdfSearch.textOutputQueue.length > 0 
						&& pdfSearch.textOutputQueue[lastIndex] != pageIdx
						&& pdfSearch.textOutputQueue[lastIndex-1] != pageIdx) {
					//console.log("span break!!!!!");
					return false;
				}
				if (elem.w <0 || elem.h<=0){
					//return true;
				}
				var elemTop = elem.t;
				var elemLeft = elem.l;
				if(typeof offsetX !== "undefined") {
					elemLeft = parseFloat(elem.l)+offsetX;
				}
				if(typeof offsetY !== "undefined") {
					elemTop = parseFloat(elem.t)+offsetY;
				}

				// 모바일에서는 링크를 기본으로 출력해준다.
				if (localSynap.properties.useMobileTextSearch == false && BROWSER.isMobile() && typeof elem.link === "undefined") {
					return;
				}
				var htmlHead = '';
				var htmlTail = '';
				if (typeof elem.link === "string") {
					htmlHead = '<a href="' + elem.link + '" target="_blank" title="' + elem.link + '">';
					htmlTail = '</a>';
				}

				html += htmlHead +
						"<span class=\"pageText\" style='" +
						"top:" + elemTop * adjustRatio + "px; " +
						"left:" + elemLeft * adjustRatio + "px; " +
						"width:" + elem.w * adjustRatio + "px; " +
						"height:" + elem.h * adjustRatio + "px; " +
						"font-size:" + elem.h * adjustRatio + "px; " +
						"position:absolute; " + 
						opacityZeroString + 
						"' ";

				// 링크태그라면 처리해준다.
				if (typeof elem.link === "number") {
					html += "onclick='localSynap.movePage("+elem.link+");'";
				} 
				// class명은 Image와 레이아웃을 맞추기 위해 동일하게 해주었다.
				if(typeof elem.text === "undefined" || elem.text=="") {
					html += ">&nbsp;</span>";
				} else {
					html += ">" + elem.text + "</span>";
				}
				html += htmlTail;
				counter++;
			});
			// 온전하게 출력이 다 되었다면, 텍스트출력큐를 초기화해준다.
			if (counter == textNodes.length) {
				pdfSearch.textOutputQueue = [];
			}
			
			return html;
		},
		getSpanObject: function(pageIdx, parsedData, offsetX, offsetY) {
			if(localSynap.properties.useMobileTextSearch == false && BROWSER.isMobile())
				return;
			var textNodes = parsedData[pageIdx];
			if(textNodes == undefined) { return; }
			// 이미지변환기용 스킨은 원본이미지와 크기가 다른 경우가 있다.
			// 텍스트를 배치할 때 그 비율이 필요하며, 아래의 코드가 사용된다. 다소 손볼곳이 많으나, 구조가 안정화 된 후에 정리하자
			if (localSynap.pxToPtRatio == undefined && localSynap.isImageMode()) {
				localSynap.pxToPtRatio = parseInt(localSynap.$pageAreas[0].css('width')) / localSynap.pageInfo.width;
				RATIO_NUMBERS.setStandardRatio(localSynap.pxToPtRatio);
			}
			var adjustRatio =  RATIO_NUMBERS.getRatio();
			var lastIndex = pdfSearch.textOutputQueue.length-1;
			var frag = document.createDocumentFragment();
			var len = textNodes.length;
			for (var idx = 0; idx < len; ++idx) {
				elem = textNodes[idx];
				// spanString을 만드는 도중, 텍스트출력큐에 내용이 들어오면
				// 다른페이지를 출력해야 할 때, 바로 종료한다.
				// 두 페이지만 텍스트출력을 허용한다.
				if (pdfSearch.textOutputQueue.length > 0 
						&& pdfSearch.textOutputQueue[lastIndex] != pageIdx
						&& pdfSearch.textOutputQueue[lastIndex-1] != pageIdx) {
					//console.log("span break!!!!!");
					return false;
				}
				var elemTop = elem.t;
				var elemLeft = elem.l;
				if(typeof offsetX !== "undefined") {
					elemLeft = parseFloat(elem.l)+offsetX;
				}
				if(typeof offsetY !== "undefined") {
					elemTop = parseFloat(elem.t)+offsetY;
				}

				// 모바일에서는 링크를 기본으로 출력해준다.
				if (localSynap.properties.useMobileTextSearch == false && BROWSER.isMobile() && typeof elem.link === "undefined") {
					return;
				}
				spanTag = document.createElement("span");
				if (typeof elem.link === "string") {
					a = document.createElement("a");
					a.href = elem.link;
					a.target = "_blank";
					a.title = elem.link;
					a.appendChild(spanTag);
					frag.appendChild(a);
				} else {
					frag.appendChild(spanTag);
				}
				spanTag.className += " pageText";

				spanTag.style.top = elemTop * adjustRatio;
				spanTag.style.left = elemLeft * adjustRatio;
				spanTag.style.width = elem.w * adjustRatio;
				spanTag.style.height = elem.h * adjustRatio;
				spanTag.style.fontSize = elem.h * adjustRatio;
				spanTag.style.position = "absolute";
				if (BROWSER.VERSION.IE() <= 8) {
					spanTag.style.filter = "alpha(opacity=0)";
				} else {
					spanTag.style.color = 'transparent';
				}

				// 링크태그라면 처리해준다.
				if (typeof elem.link === "number") {
					spanTag.onclick = "localSynap.movePage("+elem.link+");";
				}
				// class명은 Image와 레이아웃을 맞추기 위해 동일하게 해주었다.
				if(typeof elem.text === "undefined" || elem.text=="") {
					spanTag.innerHTML = "&nbsp;";
				} else {
					spanTag.innerHTML = elem.text;
				}
			}

			document.getElementById("page-area"+pageIdx).appendChild(frag.cloneNode(true));
			// 온전하게 출력이 다 되었다면, 텍스트출력큐를 초기화해준다.
			pdfSearch.textOutputQueue = [];
		},
		// 텍스트 검색 실행 함수
		search : function (text, isPrev) {
			// 인덱스 검사
			var pgIndex = localSynap.getCurrentPage()-1;
			text = text.toLowerCase();
			// 새 검색어를 찾을 때, 기존 하이라이트를 초기화 한다.
			if (pdfSearch.curSearchText !== text) {
				for (var tIndex = 0; tIndex < localSynap.pageSize ; ++tIndex) {
					
					var $pageElem = localSynap.$pageAreas[tIndex].children();
					// 자식태그가 이미지만 있을 경우는 패스
					if ($pageElem.length == 1) continue;

					for (var index = 0; index < $pageElem.length; ++index) {
						if($pageElem[index].style.backgroundColor) {
							$pageElem[index].style.backgroundColor = "transparent";
						}
					}
				}
			}

			if (text.length <= 0) return;

			// 검색 시작
			loadSpinner('contents_pdf_spinner');
			pdfSearch.curSearchText = text;
			if (pdfSearch.findList[text] === undefined) {
				pdfSearch.findList[text] = {};
				if (isPrev) {
					pdfSearch.findPageIdx = localSynap.pageSize -1;
					pdfSearch.findTextIdx = 9999999;
				} else {
					pdfSearch.findPageIdx = 0;
					pdfSearch.findTextIdx = 0;
				}
			}
			localSynap.highlightQueue = null;

			// XML을 순차탐색하여 문자열을 찾는다.
			pdfSearch.loadXmlQueue = [];
			if (isPrev) {
				for (var index = pgIndex; index >= 0; --index) {
					pdfSearch.loadXmlQueue.push(index);
				}
				for (var index = localSynap.pageSize-1; index >= pgIndex; --index) {
					pdfSearch.loadXmlQueue.push(index);
				}
			} else {
				for (var index = pgIndex; index < localSynap.pageSize; index++) {
					pdfSearch.loadXmlQueue.push(index);
				}
				for (var index = 0; index < pgIndex; index++) {
					pdfSearch.loadXmlQueue.push(index);
				}
			}
			intervalIdx = 0;
			$inputBox = $('#searchText');

			if (BROWSER.VERSION.IE() <= 9) {
				cycleTime = 10;
			} else {
				cycleTime = 1;
			}

			if (pdfSearch.cir == undefined) {
				pdfSearch.cir = setInterval(function () {
					if (intervalIdx >= localSynap.pageSize) {
						clearInterval(pdfSearch.cir);
						removeSpinner();
						////console.log("cannot find " + text);
						$inputBox.removeClass('inputIdle');
						$inputBox.addClass('inputError');
						$inputBox.focus(function () {
							if ($inputBox.hasClass('inputError')) {
								$inputBox.val(pdfSearch.curSearchText);
							}
						});
						$inputBox.click(function () {
							if ($inputBox.hasClass('inputError')) {
								$inputBox.val(pdfSearch.curSearchText);
							}
						});

						pdfSearch.cir = undefined;
						return;
					}
					idx = pdfSearch.loadXmlQueue[intervalIdx];
					////console.log("index", idx);
					pdfSearch.ret = pdfSearch.findTextInPage(idx, text, isPrev);
					if (pdfSearch.ret == true) {
						clearInterval(pdfSearch.cir);
						removeSpinner();

						if ($('#searchText').hasClass('inputError')) {
							$inputBox.removeClass('inputError');
							$inputBox.addClass('inputIdle');
						}
						pdfSearch.cir = undefined;

					} else {
						intervalIdx++;
					}
				}, cycleTime);
			}
			return;
		},
		checkSpanTag: function (){
			startTime = new Date();
			for (var pageIndex = 0; pageIndex < 10; ++pageIndex) {
				pdfSearch.getSpanObject(pageIndex, localSynap.searchMeta);
			}
			endTime = new Date();
			//console.log("elapsed : ", endTime.getTime() - startTime.getTime());
		},
		checkSpanString: function (){
			startTime = new Date();
			for (var pageIndex = 0; pageIndex < 10; ++pageIndex) {
				localSynap.$pageAreas[pageIndex].append($(pdfSearch.getSpanString(pageIndex, localSynap.searchMeta)));
			}
			endTime = new Date();
			//console.log("elapsed : ", endTime.getTime() - startTime.getTime());
		}
	};
	return Search;
})();
