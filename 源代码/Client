import net.sf.json.JSONObject;

import java.io.*;
import java.net.InetAddress;
import java.net.Socket;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.util.HashSet;
import java.util.Objects;


public class SocketClient {

    //允许上传的图片类型
    private final HashSet<String> IMG_TYPE = new HashSet<String>() {{
        add("png");
        add("jpeg");
        add("jpg");
    }};
    //允许上传的图片大小(10MB)
    private final int MAX_LENGTH = 1024 * 1024 * 10;
    //服务器IP地址
    private final String URL = "192.168.56.1";
    //服务器端口号
    private final int PORT = 9877;
    //图片路径
    private final String ImgPath = "src/4.jpeg";
    private Socket mSocket;
    private OutputStream outputStream;
    private InputStream inputStream;

    public SocketClient() {
        try {
            SendImage();
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }

    //发送图片到服务器
    public void SendImage() throws IOException, InterruptedException {
        File file = new File(ImgPath);
        //图片长度
        long imgLength = file.length();
        //图片类型
        String imgType = Files.probeContentType(file.toPath()).split("/")[1];
        System.out.println("图片大小:" + imgLength + "B;图片类型:" + imgType);
        //对文件大小及类型进行限制判断
        if (imgLength > MAX_LENGTH) {
            System.out.println("图片超出最大限制，请重新选择一张图片！");
            return;
        } else if (!IMG_TYPE.contains(imgType)) {
            System.out.println("文件类型不符合要求，请重新选择一张图片！");
            return;
        }
        //建立Socket连接
        mSocket = new Socket(InetAddress.getByName(URL), PORT);
        System.out.println("与服务器建立连接成功！");

        //获取socket输出流与输入流
        outputStream = mSocket.getOutputStream();
        inputStream = mSocket.getInputStream();

        //发送图片详细信息给服务器
        //使用JSON格式对数据进行封装
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("Length", imgLength);
        jsonObject.put("FileType", imgType);
        //将JSON格式数据转换成字节数组，使用UTF-8编码
        byte[] sendMessage = jsonObject.toString().getBytes(StandardCharsets.UTF_8);
        //发送字节数组
        outputStream.write(sendMessage);

        //接收服务器端发送过来的确认数据
        System.out.println("接收服务器的确认数据中......");
        String confirmData = readMessage();
        System.out.println("服务器确认信息:" + confirmData);
        if (!Objects.equals(confirmData, "OK")) {
            System.out.println("发送给服务器的图片信息错误");
            return;
        }
        System.out.println("接收确认消息完成......");

        //发送图片------------------
        //获取图片的缓冲输入流
        System.out.println("图片上传中......");
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file));
        int readLen;
        byte[] buf = new byte[1024];
        while ((readLen = bis.read(buf)) != -1) {
            outputStream.write(buf, 0, readLen);
        }
        mSocket.shutdownOutput();
        System.out.println("图片上传完成......");

        //接收来自服务端的回复
        String result = readMessage();
        System.out.println("服务器回传消息:" + result);

        //关闭图片缓冲输入流
        bis.close();
        //关闭socket输入流
        inputStream.close();
        //关闭socket输出流
        outputStream.close();
        //断开socket连接
        mSocket.close();
    }

    //读取服务器发送过来的消息
    private String readMessage() {
        try {
            byte[] bytes = new byte[1024];
            int len = inputStream.read(bytes);
            return new String(bytes, 0, len);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static void main(String[] args) {
        new SocketClient();
    }
}
