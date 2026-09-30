# [forensics] you cut me off
<img width="1594" height="972" alt="Screenshot 2026-09-30 at 18 18 27" src="https://github.com/user-attachments/assets/336897ca-bff8-4f7a-b9d9-978327beb27a" />

문제에서 제공된 이미지 파일을 뜯어보는 게 제일 빠를 것 같다. png 파일이니까 스테가노그라피 관련한 문제이지 않을까..싶다. 




<img width="492" height="382" alt="hereyougo" src="https://github.com/user-attachments/assets/7c5a3a02-2813-412c-8c22-9e9538869691" />

항상 png 파일 문제를 풀 때는 hex editor로 파일이 여러 개가 숨겨져 있지는 않은지, 헤더는 png인데 푸터 쪽은 jpg로 끝나지는 않는지를 먼저 보는 편이다. 이번에도 가볍게 훑어주었다.


<img width="892" height="86" alt="Screenshot 2026-09-30 at 18 21 47" src="https://github.com/user-attachments/assets/b26dbdc2-3e55-4587-b97a-ee40572623da" />
<img width="1418" height="470" alt="Screenshot 2026-09-30 at 18 21 28" src="https://github.com/user-attachments/assets/fe5dafad-2ecb-46cb-ab5e-d2d310ddf8e8" />

시그니처도 한 쌍밖에 없어서 우선 가볍게 훑었을 땐 큰 문제가 안 보인다. 그런데 이미지를 잘 살펴보면, 묘하게 아랫 부분이 잘려있는 듯한 느낌이 온다. 




<img width="492" height="382" alt="hereyougo" src="https://github.com/user-attachments/assets/9baf6547-6468-42a9-8a43-fe7fbb1a9017" />

화살표 친 부분에 반투명한 박스도 보이고, ill type it out<< 이 문구를 봤을 때도 분명히 다음에 뭔갈 쳐서 보내줬을 것 같은 뉘앙스다. 이미지 세로 크기를 늘려보면 어떨까?




<img width="1456" height="142" alt="Screenshot 2026-09-30 at 18 44 13" src="https://github.com/user-attachments/assets/eb9832c7-b5c9-4689-9ec7-9c10fa234354" />
우선 지금 사이즈는 아래 사진과 같이 492x380이다. 만약에 크기를 바꿔주면 CRC값이 바뀌어서 이건 pngcheck로 수정을 해주어야 한다. 넉넉하게 높이를 2000으로 수정 해주자.<br></br>



<img width="256" height="108" alt="Screenshot 2026-09-30 at 18 47 33" src="https://github.com/user-attachments/assets/520e7d9c-85b4-4d49-8aec-ba596c0af501" />

16진수로 2000은 7D0이다. 



<img width="948" height="178" alt="Screenshot 2026-09-30 at 19 07 48" src="https://github.com/user-attachments/assets/570faa90-5e6d-401e-8edb-c08e73b0f3af" />

높이를 2000으로 수정해주었다. 이 상태 그대로 저장하면 빨간색 박스를 친 crc값이 오류가 발생하니, pngcheck로 정상 crc 값을 구해주자.


<img width="1092" height="92" alt="Screenshot 2026-09-30 at 19 07 55" src="https://github.com/user-attachments/assets/14a17ccd-0573-44a8-aff9-31a36759496b" />
<img width="944" height="98" alt="Screenshot 2026-09-30 at 19 08 05" src="https://github.com/user-attachments/assets/e79ab596-69a9-40f4-83dd-2c5b285ed7e7" />

높이값도, crc 값도 잘 수정이 되었으니 이제 사진이 잘 열릴 것이다. 


<img width="816" height="704" alt="Screenshot 2026-09-30 at 19 08 18" src="https://github.com/user-attachments/assets/d774f1c3-f35e-4ba1-9d39-f83952483ac9" />

내 예상이 맞았다~~~!! 플래그가 바로 보인다. 




<img width="1532" height="1066" alt="Screenshot 2026-09-30 at 19 08 35" src="https://github.com/user-attachments/assets/4652a970-ebfd-477c-afc1-7fd31837a054" />

문제 풀이 성공! 다만 플래그까지 다 구하고 느낀 건데, 문제에서부터 힌트가 있었다. **You cut me off** 이름 자체가 딱 봐도 사진을 잘라서 낸 문제인 티가 팍팍 났었다..
















