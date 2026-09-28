# [forensics] Welcome Call!
<img width="1588" height="958" alt="Screenshot 2026-09-29 at 02 36 00" src="https://github.com/user-attachments/assets/6df4d455-3d68-4074-b06b-803e45ab4291" />

문제 설명은 딱히 풀이와 연관은 없어보인다. flag factory에서 최애 CTFer를 찾는다고 한다. 우선 첨부된 pcap 파일을 와이어샤크로 열어봐야겠다. 



<img width="2172" height="1816" alt="Screen Shot 2026-09-29 at 05 05 53 AM" src="https://github.com/user-attachments/assets/dda76f14-a702-41b1-a2fd-f0a84e0fa804" />

우선 SIP, RTP 프로토콜이 눈에 띈다. 전공 수업 시간에 들어본 개념이지만, 가물가물해져서 이번 기회를 통해 간단하게 복습해보자.
- **SIP(Application Layer)**: Session Initation Protocol. 멀티미디어 통신에 있어서 세션이나 call을 관리하는 프로토콜로, 데이터 전송이 아니라 **시그널링을 통한 통신 관리** 역할을 한다. 
- **RTP(Application Layer)**: Real-time Transport Protocol. 실시간 전송 프로토콜로, 비디오 및 오디오 **데이터를 일정한 포맷으로 계속 전송**한다. 신속하게 데이터를 보내는 게 핵심이니, UDP 레이어를 이용한다. 


<img width="850" height="840" alt="image" src="https://github.com/user-attachments/assets/488ba175-ca82-4d82-a646-d663a1605c65" />


**<위 이미지를 통한 SIP, RTP 동시 정리>**
- **1단계(연결부)**: SIP 사용. 발신자가 통화 요청을 위해 Invite 메시지를 서버로 보냄. 서버는 수신자에게 메시지를 전달하고, 발신자한테는 처리 중임을 알리기 위해 **100 trying**을 보냄. 수신자에게 전화가 가면 **180 ringing** 메시지가 발신자한테 전달됨. 전화가 연결되면 수신자가 받았음을 의미하는 **200 OK** + 발신자가 확인했다는 **ACK**로 마무리.
- **2단계(real 음성 통신부)**: 실제 음성 데이터 전송이 필요하니까 이 단계부터는 RTP가 사용됨.
- **3단계(종료부)**: SIP 사용. 통화를 끊는 쪽이 Bye를 보내고, 상대방이 200 OK로 응답하면 종료됨.



이제 다시 문제 풀이로 넘어오자. 항상 와이어샤크에서 문제를 풀 때는 Follow → 특정 Stream을 확인하는 게 나만의 루틴(?)이라서 이번에도 해보았다. 문제 풀이에 유의미할지는 모르겠으나, SIP 패킷이 어떤지 텍스트로 볼 수 있었다.


<img width="3072" height="1800" alt="Screen Shot 2026-09-29 at 05 19 50 AM" src="https://github.com/user-attachments/assets/b82e4527-95e8-4d86-8dc6-1696cab7f2f2" />

발신자 / 수신자 IP를 볼 수 있고, 통화가 연결되는 흐름을 직접 확인할 수 있었다. 



위에서는 플래그와 관련된 정보를 얻을 수 없었으니, 이제 RTP를 분석해야 할 차례다. 구글링을 해보면 와이어샤크에서는 **Telephony → RTP Stream**를 들어가면 직접 음성 데이터를 들을 수 있다고 한다.



<img width="2244" height="1304" alt="Screen Shot 2026-09-29 at 05 20 46 AM" src="https://github.com/user-attachments/assets/85f52741-e0aa-4bf7-9cea-f88373881405" />
여기서 Play Streams를 누르니 뭔가 역재생이 된 것 같은 녹음본(?)을 들을 수 있었다. 다시 역재생을 해서 편집하면 힌트를 얻을 수 있을 것 같은 확신이 들어서, Export하여 역재생 편집을 해주었다.




<img width="1420" height="310" alt="Screenshot 2026-09-29 at 05 50 48" src="https://github.com/user-attachments/assets/c47b51b6-9602-4dec-aef6-ec41576d0b5e" />
예상대로, 플래그를 목소리로 직접 말해주는 걸 들을 수 있었다. sun{thankyouforplaying}이라고 한다.




<img width="1524" height="1064" alt="Screenshot 2026-09-29 at 05 15 35" src="https://github.com/user-attachments/assets/544c4be7-3fd7-4d23-a11f-399983ff9fa2" />
성공!





  
