```python
import sys
from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QHBoxLayout, QTextEdit, QLineEdit, QPushButton, QLabel
import openai
import json
import os
import requests
from PyQt5.QtGui import QFont
from PyQt5.QtWidgets import QGraphicsBlurEffect
from PyQt5.QtGui import QPixmap
from PyQt5.QtCore import Qt

openai.api_key = "your-api-key"


class ChatGPT:
    def __init__(self, user):
        self.user = user
        self.messages = [{"role": "system", "content": "一个有10年Python开发经验的资深算法工程师"}]
        self.filename = "./user_messages.json"

    def ask_gpt(self):
        # q = "用python实现：提示手动输入3个不同的3位数区间，输入结束后计算这3个区间的交集，并输出结果区间"
        rsp = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=self.messages
        )
        return rsp.get("choices")[0]["message"]["content"]

    def writeTojson(self):
        try:
            # 判断文件是否存在
            if not os.path.exists(self.filename):
                with open(self.filename, "w") as f:
                    # 创建文件
                    pass
            # 读取
            with open(self.filename, 'r', encoding='utf-8') as f:
                content = f.read()
                msgs = json.loads(content) if len(content) > 0 else {}
            # 追加
            msgs.update({self.user: self.messages})
            # 写入
            with open(self.filename, 'w', encoding='utf-8') as f:
                json.dump(msgs, f)
        except Exception as e:
            print(f"错误代码：{e}")


def getmessage(q):
    user = "My"
    chat = ChatGPT(user)
    # 循环
    while 1:
        # 限制对话次数
        if len(chat.messages) >= 11:
            print("******************************")
            print("*********强制重置对话**********")
            print("******************************")
            # 写入之前信息
            chat.writeTojson()
            user = input("请输入用户名称: ")
            chat = ChatGPT(user)

        # 提问
        # 逻辑判断
        if q == "0":
            print("*********退出程序**********")
            # 写入之前信息
            chat.writeTojson()
            break
        elif q == "1":
            print("**************************")
            print("*********重置对话**********")
            print("**************************")
            # 写入之前信息
            chat.writeTojson()
            user = input("请输入用户名称: ")
            chat = ChatGPT(user)
            continue

        # 提问-回答-记录
        chat.messages.append({"role": "user", "content": q})
        answer = chat.ask_gpt()
        chat.messages.append({"role": "assistant", "content": answer})
        return answer
#》？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？以上是AI对话区？？？？？？？？？？？？？？？？？？
class ChatWidget(QWidget):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        #毛玻璃效果
        self.bg = QLabel(self)
        response = requests.get('http://blog.isimp.xyz/wp-content/uploads/2023/04/f50f14e28dff7964f16f6f83328bf5c1-scaled.jpg', headers={
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3'},
                                timeout=30)
        pixmap = QPixmap()
        if response.content:
            pixmap.loadFromData(response.content)
        blur_effect = QGraphicsBlurEffect()
        blur_effect.setBlurRadius(20)
        self.bg.setGraphicsEffect(blur_effect)
        self.bg.resize(self.size())
        self.bg.setPixmap(pixmap.scaled(self.width(), self.height()))
        self.setWindowFlags(Qt.FramelessWindowHint)
        self.setAttribute(Qt.WA_TranslucentBackground)
        # 将关闭按钮添加到水平布局中
        title_layout = QHBoxLayout()
        title_layout.addStretch(1)
        close_button = QPushButton('×', self)
        close_button.setFixedSize(15, 15)
        close_button.clicked.connect(self.close)
        close_button.setStyleSheet("""
                    background-color: skyblue;
                    border-radius: 10px;
                    color:white;
                    font-size:10px;
                """)
        title_layout.addWidget(close_button)
        # 聊天记录显示区域
        self.chat_history = QTextEdit()
        self.chat_history.setReadOnly(True)
        self.chat_history.setFont(QFont('SimHei', 12))
        # 消息输入框和发送按钮
        self.msg_input = QLineEdit()
        self.msg_input.setFont(QFont('SimHei', 12))
        self.send_btn = QPushButton('Send')
        self.send_btn.clicked.connect(self.on_send_btn_clicked)

        # 设置 input_box 的快捷键为 Enter 键，并连接到 send_message 槽函数上
        self.msg_input.returnPressed.connect(self.on_send_btn_clicked)
        # 布局设置
        input_layout = QHBoxLayout()
        input_layout.addWidget(self.msg_input)
        input_layout.addWidget(self.send_btn)

        main_layout = QVBoxLayout()
        main_layout.addWidget(self.chat_history)
        main_layout.addLayout(input_layout)

        self.setLayout(main_layout)

    def on_send_btn_clicked(self):
        msg_text = self.msg_input.text()
        self.msg_input.clear()

        reply_text = getmessage(msg_text)
        self.chat_history.append(f'You: {msg_text}')
        self.chat_history.append(f'System: {reply_text}')






if __name__ == '__main__':
    app = QApplication(sys.argv)
    chat_widget = ChatWidget()
    chat_widget.show()
    sys.exit(app.exec_())
```