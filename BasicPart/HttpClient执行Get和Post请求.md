HttpClient执行Get和Post请求
===

1. Get

    ```java
	/**
     * 采用httpclient的方式 用get提交数据到服务器
     */
    public void loginByClientGet(View view) {
        String password = et_password.getText().toString().trim();
        String name = et_username.getText().toString().trim();
        if (TextUtils.isEmpty(name) || TextUtils.isEmpty(password)) {
            Toast.makeText(this, "用户名密码不能为空", 1).show();
            return;
        }
        // 1.打开浏览器
        HttpClient client = new DefaultHttpClient();
        // 2.输入浏览器的地址
        String uri = "http://192.168.1.100:8080/web/LoginServlet?" + "name="
                + URLEncoder.encode(name) + "&password="
                + URLEncoder.encode(password);
        HttpGet httpGet = new HttpGet(uri);
        // 3.敲回车.
        try {
            HttpResponse response = client.execute(httpGet);
            int code = response.getStatusLine().getStatusCode();
            if (code == 200) {
                InputStream is = response.getEntity().getContent();
                String result = StreamTools.readFromStream(is);
                Toast.makeText(this, result, 1).show();
            } else {
                Toast.makeText(this, "服务器异常", 1).show();
            }
        } catch (Exception e) {
            e.printStackTrace();
            Toast.makeText(this, "访问网络异常", 1).show();
        }
    }
    ```
2. Post

    ```java
    /**
     * 采用httpclient post数据到服务器
     */
    public void loginByClientPost(View view) {
        String password = et_password.getText().toString().trim();
        String name = et_username.getText().toString().trim();
        if (TextUtils.isEmpty(name) || TextUtils.isEmpty(password)) {
            Toast.makeText(this, "用户名密码不能为空", 1).show();
            return;
        }
        try {
            // 1.创建一个浏览器
            HttpClient client = new DefaultHttpClient();
            // 2.准备一个连接
            HttpPost post = new HttpPost(
                    "http://192.168.1.100:8080/web/LoginServlet");
            // 要向服务器提交的数据实体
            List<NameValuePair> parameters = new ArrayList<NameValuePair>();
            parameters.add(new BasicNameValuePair("name", name));
            parameters.add(new BasicNameValuePair("password", password));
            UrlEncodedFormEntity entity = new UrlEncodedFormEntity(parameters,
                    "utf-8");
            post.setEntity(entity);
            // 3.敲回车
            HttpResponse response = client.execute(post);
            int code = response.getStatusLine().getStatusCode();
            if (code == 200) {
                InputStream is = response.getEntity().getContent();
                String result = StreamTools.readFromStream(is);
                Toast.makeText(this, result, 1).show();
            } else {
                Toast.makeText(this, "服务器异常", 1).show();
            }
        } catch (Exception e) {
            e.printStackTrace();
            Toast.makeText(this, "client post 失败", 1).show();
        }
    }
    ```
	
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 