
联众打码平台接入
local softwareId = 15137
local softwareSecret = 'bGlmSX0tEz0BfIObuzitna5tLMq6aynfxe3MBBmS'
local lianzhong = {
	ocr = function(user, pass, img, typeid)
		-- 自适应图片对象 或 文件路径
		local img_base64_data
		if image.is(img) then
			img_base64_data = img:png_data():base64_encode()
		elseif file.exists(img) then
			img_base64_data = file.reads(img):base64_encode()
		else
			error('传入第三项值非图像对象或图像路径', 2)
		end
		
		local code, header, content = http.post(
			'https://v2-api.jsdama.com/upload',
			Timeout or 10,
			{
				['Host'] = 'v2-api.jsdama.com',
				['Connection'] = 'keep-alive',
				['Accept'] = 'application/json, text/javascript, */*; q=0.01',
				['User-Agent'] = 'XXTouch',
				['Content-Type'] = 'text/json',
			},
			json.encode({
				softwareId = softwareId,
				softwareSecret = softwareSecret,
				username = user,
				password = pass,
				captchaData = img_base64_data,
				captchaType = typeid,
				captchaMinLength = 0,
				captchaMaxLength = 0,
				workerTipsId = 0
			})
		)
		if code == 200 then
			local jobj = json.decode(content)
			if not jobj then return false, "无法解析的内容" end
			if jobj.code == 0 then
				return jobj.data.recognition, jobj.data.captchaId
			else
				return false, jobj.message
			end
		else
			return nil, '错误：超时'
		end
	end,
	report_error = function(user, pass, resultid)
		local code, header, content = http.post(
			'https://v2-api.jsdama.com/report-error',
			Timeout or 10,
			{
				['Host'] = 'v2-api.jsdama.com',
				['Connection'] = 'keep-alive',
				['Accept'] = 'application/json, text/javascript, */*; q=0.01',
				['User-Agent'] = 'XXTouch',
				['Content-Type'] = 'text/json',
			},
			json.encode({
				softwareId = softwareId,
				softwareSecret = softwareSecret,
				username = user,
				password = pass,
				captchaId = resultid
			})
		)
		if code == 200 then
			local jobj = json.decode(content)
			if not jobj then return false, "无法解析的内容" end
			if jobj.code == 0 then
				return jobj.data.result
			else
				return false, jobj.message
			end
		else
			return nil, '错误：超时'
		end
	end,
	get_point = function(user, pass)
		local code, header, content = http.post(
			'https://v2-api.jsdama.com/check-points',
			Timeout or 10,
			{
				['Host'] = 'v2-api.jsdama.com',
				['Connection'] = 'keep-alive',
				['Accept'] = 'application/json, text/javascript, */*; q=0.01',
				['User-Agent'] = 'XXTouch',
				['Content-Type'] = 'text/json',
			},
			json.encode({
				softwareId = softwareId,
				softwareSecret = softwareSecret,
				username = user,
				password = pass,
				captchaId = ID
			})
		)
		if code == 200 then
			local jobj = json.decode(content)
			if not jobj then return false, "无法解析的内容" end
			if jobj.code == 0 then
				return jobj.data
			else
				return false, jobj.message
			end
		else
			return nil, '错误：超时'
		end
	end
}

--上面为联众接口,下面我们开始直接调用,以kindle为例
function tab.KindleIdentificationcode(x1, y1, x2, y2, typeid) --识别验证图片 x1,y1 图片左顶点  x2, y2 图片右底点 typeid 识别类型
	local lz = lianzhong -- 加载模块
	local user = 'terminate'
	local pass = 'zhangbi0126..'
	local img = screen.image(x1, y1, x2, y2)
	local typeid = typeid
	local Result=''
	-- 上传图?信息同时获取结果
	Result=lz.ocr(user, pass, img, typeid)
	while Result == '' and Result == nil do
		sys.log('等待验证码传输中')	
		sys.msleep(500)
	end
	return Result
End

--接口里面封装了与打码平台的交互方式,调用时传入他认识的参数即可,截图转base64然后post上传至联众平台,联众平台解析base64后识别图片,返回JSON格式的
--string类型数据给你
