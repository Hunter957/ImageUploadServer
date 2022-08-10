import socket
import time
import json
from threading import Thread

# 服务器主机地址和监听的端口号
HOST = socket.gethostname()
PORT = 9877


# 服务端回应消息
def response_message():
    message = 'I have receive your image'
    return message.encode("utf-8")


# 接收图片数据
def recv_img(client_sock):
    data = b''
    print("开始接收客户端数据--------")
    try:
        confirm_data = client_sock.recv(1024)
        print(confirm_data.decode('utf-8'))
        json_data = json.loads(confirm_data)
        img_type = json_data['FileType']
        img_length = json_data['Length']
        client_sock.send('OK'.encode('utf-8'))
        print('发送确认消息完成--------')
    except KeyError as a:
        print("图片信息数据错误:" + str(a))
        client_sock.send('ERROR')
        return

    print("开始接收图片数据---------------")
    while len(data) < img_length:
        try:
            recv_data = client_sock.recv(1024000)
            time.sleep(0.1)
            data += recv_data
        except Exception as e:
            print("Socket接收数据错误:" + str(e))
            break
    if len(data) != 0:
        save_image(data, img_type)
    else:
        print("Client Close!")


# 将传输过来的图片保存到img目录下
def save_image(imageData, imgType):
    _time = time.strftime("%Y%m%d_%H%M%S", time.localtime())
    imageSavePath = "img/" + _time + "." + imgType
    print(imageSavePath)
    file = open(imageSavePath, 'wb+')
    file.write(imageData)
    file.close()


# 处理客户端请求过来的数据
def accept_data(client_sock, client_addr):
    recv_img(client_sock)
    # 生成回复消息
    response = response_message()
    # 发送服务器回应消息
    client_sock.send(response)
    # 关闭与客户端的socket连接
    client_sock.shutdown(1)
    client_sock.close()


# Socket 服务器类
class SocketServer:
    def __init__(self, host=HOST, port=PORT):
        self.sock = None
        self.host = host
        self.port = port
        self.setup_socket()
        self.accept()
        self.shout_down_socket()

    # 建立Socket连接
    def setup_socket(self):
        # 创建 socket 对象
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # 绑定端口号
        self.sock.bind((self.host, self.port))
        # 设置最大连接数，超过后排队
        self.sock.listen(600)

    # 使用多线程接受客户端的请求
    def accept(self):
        while True:
            # address ----> 消息发送方ip地址
            # client ----> 服务器与客户端之间的Socket连接
            (client, address) = self.sock.accept()
            # target ----> 线程执行的方法 ;    args ----> target目标调用的参数元组
            th = Thread(target=accept_data, args=(client, address))
            th.start()

    # 关闭Socket连接
    def shout_down_socket(self):
        if self.sock is not None:
            self.sock.shutdown()
            self.sock.close()


if __name__ == "__main__":
    SocketServer()
