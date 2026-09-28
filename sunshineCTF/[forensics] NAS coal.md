# [forensics] NAS coal

<img width="1584" height="888" alt="nas_coal1" src="https://github.com/user-attachments/assets/6a017d87-35bb-43d9-a4c5-bbea96a45585" />

누군가 보석 컬렉션에 석탄을 넣어놨다고 한다. 우선 첨부된 파일을 봐야 어떻게 풀이를 해야할지 감이 올 것 같다.


<img width="3424" height="2024" alt="Screen Shot 2026-09-29 at 04 19 42 AM" src="https://github.com/user-attachments/assets/615c658f-e024-4575-8645-b110262970d0" />

PPTM 파일이라서 프레젠테이션이 보인다. 슬라이드 속 이미지들을 훑어보니, base64인가..? 싶은 이미지가 한 장 있다.


<img width="2072" height="1064" alt="Screenshot 2026-09-29 at 04 19 47" src="https://github.com/user-attachments/assets/03a56f72-72ec-4a6b-a5fb-c352ad553a0c" />

= 기호가 없어서 애매하기는 하지만, 일단 base64 decode를 돌려보면 어떨까. 사진 순서대로 조합해서 돌려보자.


<img width="602" height="916" alt="Screenshot 2026-09-29 at 04 23 36" src="https://github.com/user-attachments/assets/652e4326-7dc3-457b-8324-b453bdbec230" />

유효한 단어나 문장이 안 나오는 걸 보니 방향을 잘못 짚은 것 같다. 


<img width="2246" height="1268" alt="Screenshot 2026-09-29 at 04 19 53" src="https://github.com/user-attachments/assets/4998ba5f-d45f-4c3e-b75f-1d0c8f58fa17" />

슬라이드를 다시 쭉 훑어보다가, 처음 보는 단어인 **olevba**가 있었다. 찾아보니, 포렌식 문제를 풀이할 때 자주 사용되는 툴로 Microsoft Office 파일 속 매크로를 뜯어준다고 한다. 구글링해보니, word 파일만 던져주는 문제에서 이 도구를 사용해서 풀이를 하는 경우가 꽤 있었다. 무엇보다 이 문제에서는 **파일 확장자가 pptm → 매크로 포함하고 있는 ppt**라는 걸 대놓고 알려줘서, 이 툴을 알고 있는 사람이라면 훨씬 빠르게 풀이 로직을 짤 수 있었을 듯하다.



슬라이드에 mfw olevba oneshot chall이라고 적힌 걸 보니, 저 툴을 쓰면 플래그를 바로 딸 수 있을 것 같다.


``
sudo -H pip install -U oletools 
``을 써서 다운을 받아주고,
``
olevba gem_collection.pptm
``을 해주니 아래와 같은 결과값이 나왔다. 


<img width="1720" height="1058" alt="Screenshot 2026-09-29 at 04 32 50" src="https://github.com/user-attachments/assets/ded70c49-209c-4d27-b8f1-7c48eb672b5a" />

박스친 부분을 base64 decode 돌려주면 플래그가 나오겠다는 확신이 들었다.



<img width="1744" height="1158" alt="Screenshot 2026-09-29 at 04 38 41" src="https://github.com/user-attachments/assets/fa6b0b4f-0377-48ca-a18f-a846fda74f53" />
<img width="1482" height="266" alt="Screenshot 2026-09-29 at 04 39 09" src="https://github.com/user-attachments/assets/f13fdb67-b465-48c7-bdd0-d5b74872f6b7" />


성공!

**flag: sun{yup_issa_gem}**
