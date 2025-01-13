# 沃尔玛CK捕获v1.0.6


# 【重要】请先安装mitmproxy-11.0.1-windows-x86_64-installer.exe，然后在运行CK捕获

```python
def request(flow: http.HTTPFlow):
    """
    捕获请求中的 sign 值
    """
    if TARGET_URL_PATTERN in flow.request.url:
        try:
            # 检查请求类型是否为 JSON
            content_type = flow.request.headers.get('content-type', '')
            if 'application/json' not in content_type.lower():
                return
                
            # 解析请求体 JSON
            request_json = json.loads(flow.request.text)
            
            # 检查是否存在 sign 键
            if 'sign' not in request_json:
                return
                
            # 获取 sign 值并验证格式
            sign = request_json['sign']
            if not re.match(r'^[a-f0-9]{32}@[a-f0-9]{32}$', sign):
                return
                
            # 检查是否是新的CK
            if sign in captured_cks:
                ctx.log.info(f"\\n已存在的CK: {sign}")
                return
                
            # 添加到已捕获集合
            captured_cks.add(sign)
            
            # 获取当前时间
            current_time = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
            
            # 保存结果
            result = f"[{current_time}] {sign}\\n"
            output_file = os.path.join(os.getcwd(), "沃尔玛ck.txt")
            
            with open(output_file, "a+", encoding='utf-8') as f:
                f.write(result)
            
            # 写入临时文件以通知主程序
            temp_file = os.path.join(os.getcwd(), "walmart_temp.txt")
            with open(temp_file, "w", encoding='utf-8') as f:
                f.write(sign)
            
            ctx.log.info(f"\\n成功捕获新的CK\\n{sign}")
            
        except Exception as e:
            ctx.log.error(f"[-] 解析请求失败: {str(e)}")

```
