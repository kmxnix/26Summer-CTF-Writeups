# [web] Used Goods of Tomorrow

<img width="1590" height="940" alt="Screenshot 2026-09-29 at 02 35 09" src="https://github.com/user-attachments/assets/ca97af79-5baf-44ec-93f9-4ed681a12be7" />

매우 비싼 물품 구매에 성공하면 플래그를 따는 형식인 것 같다. 우선 사이트를 들어가서 어떤 흐름으로 구매가 이루어지는지 확인해봐야겠다.




<img width="1550" height="1148" alt="Screenshot 2026-09-30 at 19 25 20" src="https://github.com/user-attachments/assets/f0ef81b7-260e-4ff4-9e24-2e090b2fbc81" />

우선 계정을 만들어서 장바구니에 넣어보았다. 방법이 크레딧을 엄청 늘려서 사거나, 아니면 저 쿠폰 기능을 어찌저찌 잘 쓰면 살 수 있을 것 같다. 그러나 사이트를 둘러봤을 때 크레딧을 변경하는 부분이 안 보여서, 아마 결제할 때나 쿠폰 기능을 노려봐야 할 듯하다. 아직 명확하게 감이 안 오니까 Inspect를 써서 살펴보자.



<img width="1252" height="998" alt="Screenshot 2026-09-30 at 19 44 41" src="https://github.com/user-attachments/assets/757d0b02-bb38-4ca9-93af-ca38e639bb70" />

app.js를 뜯어보면 단서를 얻을 수 있을 듯하다. 그리고 그 전에 결제가 어떻게 돌아가는지(?) 느낌을 알고 싶어서, network 탭을 열고 구매 버튼을 눌러보았다.


<img width="2878" height="1394" alt="Screenshot 2026-09-30 at 19 46 42" src="https://github.com/user-attachments/assets/f3209b56-6caa-4036-9f21-17a89ef6e5df" />
<img width="1074" height="246" alt="Screenshot 2026-09-30 at 19 55 03" src="https://github.com/user-attachments/assets/946b334e-d5ae-4d35-b8f1-acefcb6e1d3e" />
<img width="1710" height="350" alt="Screenshot 2026-09-30 at 19 54 59" src="https://github.com/user-attachments/assets/8edffdfc-1b28-4b63-8b96-9e068e1178eb" />

여기서는 서버가 뭐라고 대답을 했는지 정도만 볼 수 있어서 >>flag<<가 어디서 반환될지에 대한 힌트만 대강 얻을 수 있었다. 이제 어떻게 값이 전달이 되는지 payload를 통해 살펴보자. 




<img width="996" height="278" alt="Screenshot 2026-09-30 at 19 52 00" src="https://github.com/user-attachments/assets/bd2553cc-e9ec-4398-b1e6-aeb2575bdce2" />
<img width="1556" height="260" alt="Screenshot 2026-09-30 at 19 51 53" src="https://github.com/user-attachments/assets/29975cf3-0d25-465e-b3e6-ea180c54f809" />

상품 Id랑 promocode가 전달되는 흐름이 보인다. 근데 여기서 Mutation<< 이라는 게 정확히 뭔지 모르겠어서 찾아봤다. 그러다가 GraphQL 취약점 분석에 대한 글을 보았고, 노출되면 안되는 구조 중에 하나라고 한다. 즉, 얻게된 단서는 이 서비스가 GraphQL 기반이라는 것이고 mutation이 아래와 같은 역할을 한다는 것이다.


**mutation 역할 in GraphQL**
- 데이터를 변경하기 위한 요청으로, POST/DELETE/PUT과 유사하다.


아직 이것만 봐서는 뭘 할지 감이 안 오니, sources에서 js 파일을 확인해봐야겠다.

```js
const TM = {
    TOKEN_KEY: 'tm_token',
    USER_KEY: 'tm_user',
    CART_KEY: 'tm_cart',

    /* --- auth token --- */
    getToken() { return localStorage.getItem(this.TOKEN_KEY); },
    getUser() { return localStorage.getItem(this.USER_KEY); },
    setAuth(token, user) {
        localStorage.setItem(this.TOKEN_KEY, token);
        localStorage.setItem(this.USER_KEY, user);
    },
    clearAuth() {
        localStorage.removeItem(this.TOKEN_KEY);
        localStorage.removeItem(this.USER_KEY);
    },

    /* --- shopping cart --- */
    getCart() {
        try { return JSON.parse(localStorage.getItem(this.CART_KEY)) || []; }
        catch (e) { return []; }
    },
    setCart(ids) { localStorage.setItem(this.CART_KEY, JSON.stringify(ids)); },
    addToCart(id) {
        const cart = this.getCart();
        cart.push(String(id));
        this.setCart(cart);
    },
    removeFromCart(index) {
        const cart = this.getCart();
        cart.splice(index, 1);
        this.setCart(cart);
    },
    clearCart() { localStorage.removeItem(this.CART_KEY); },

    /* --- GraphQL API --- */
    async gql(query, variables) {
        const headers = { 'Content-Type': 'application/json' };
        const token = this.getToken();
        if (token) headers['Authorization'] = 'Bearer ' + token;
        const res = await fetch('/graphql', {
            method: 'POST',
            headers,
            body: JSON.stringify({ query, variables: variables || {} }),
        });
        return res.json();
    },

    /* --- little corner popup notification --- */
    toast(message) {
        let el = document.getElementById('tm-toast');
        if (!el) {
            el = document.createElement('div');
            el.id = 'tm-toast';
            el.className = 'toast';
            document.body.appendChild(el);
        }
        el.textContent = message;
        // restart the show/hide animation even on rapid re-clicks
        el.classList.remove('show');
        void el.offsetWidth;
        el.classList.add('show');
        clearTimeout(this._toastTimer);
        this._toastTimer = setTimeout(() => el.classList.remove('show'), 2600);
    },

    /* --- header account strip + cart badge --- */
    async renderHeader() {
        const strip = document.getElementById('account-strip');
        if (!strip) return;
        const cartCount = this.getCart().length;
        const token = this.getToken();

        if (!token) {
            strip.innerHTML =
                '<span>Not signed in &mdash; <a class="strip-link" href="/account">open a FutureBank account</a></span>' +
                '<span><a class="strip-link" href="/checkout">🛒 Cart (' + cartCount + ')</a></span>';
            return;
        }

        // Signed in: fetch a fresh balance through the API.
        let handle = this.getUser() || 'shopper';
        let balance = '—';
        try {
            const data = await this.gql('{ myAccount { handle balanceCredits } }');
            if (data && data.data && data.data.myAccount) {
                handle = data.data.myAccount.handle;
                balance = data.data.myAccount.balanceCredits.toLocaleString();
            } else {
                // token no longer valid
                this.clearAuth();
                return this.renderHeader();
            }
        } catch (e) { /* offline; show cached */ }

        strip.innerHTML =
            '<span>FUTUREBANK: <span class="hl">' + handle + '</span> &middot; ' +
                '<span class="hl">' + balance + '</span> credits</span>' +
            '<span>' +
                '<a class="strip-link" href="/checkout">🛒 Cart (' + cartCount + ')</a> &middot; ' +
                '<a class="strip-link" href="#" id="signout">Sign out</a>' +
            '</span>';

        const so = document.getElementById('signout');
        if (so) so.addEventListener('click', (e) => {
            e.preventDefault();
            this.clearAuth();
            this.renderHeader();
        });
    },
};

document.addEventListener('DOMContentLoaded', () => TM.renderHeader());
```
확실히 자바스크립트 코드를 보면 알 수 있는 흐름이 많은 것 같다. 우선 TM.gql() 함수로 /graphql로 POST 요청을 보낸다. 그리고 인증 관련 토큰을 가져와서 인증 헤더에 넣는 로직으로 돌아간다. POST 요청을 보내는 게 TM.gql()이라는 걸 알았으니 이제 이 부분을 공략해서 /graphql이 뭘 하는지 더 찾아보자.



GraphQL에는 서버의 기능을 조회하는 >>introspection<<이라는 기능이 있다고 한다. 이걸 통해서 쿼리랑 mutation을 확인해 보자.


<img width="910" height="364" alt="Screenshot 2026-09-30 at 20 26 45" src="https://github.com/user-attachments/assets/ea41d095-cea9-44f1-8235-7b15ffba0bcb" />

그 전에, ``await TM.gql(`
  {
    myAccount {
      handle
      balanceCredits
    }
  }``을 보내서 콘솔에서 응답이 잘 돌아가는지 체크해주었다. JS 서버 응답이 올 때까지 기다리고, 이전에 확인했던 쿼리인 myAccount를 사용해서 확실하게 작동이 되는지의 여부를 봐준 것이다. 반환된 결과를 보면 계정명과 남은 잔액이 잘 뜨는 것이 보인다. 



  **<왜 쿼리랑 mutation을 봐줘야 할까..?>**
  
  웹 문제는 아직 배워가야 할 게 많아서, 이 문제를 풀 때는 어쩔 수 없이 AI의 도움을 받게 됐다. 그래서 왜 이걸 확인해줘야 하고, 왜 다음 단계 힌트로 해당 아이디어를 제시해줬는지 하나하나 봐주는 게 중요하다.
  - GraphQL에서 쿼리: 서버의 데이터를 보여달라는 요청. 단순 데이터 조회용이라고 보면 된다. 
  - "에서 mutation: 서버의 데이터를 변경하거나, 특정 작업을 실행하는 요청 → 위에서 mutation이 POST, PUT, DELETE와 비슷한 역할을 한다고 언급했던 이유기도 하다. 



  이제 쿼리 목록을 확인해 줄 차례다. 
<img width="1170" height="262" alt="Screenshot 2026-09-30 at 21 30 10" src="https://github.com/user-attachments/assets/cbe549fa-2edc-44b4-aaf2-4074bdb9abd5" />
<img width="1356" height="258" alt="Screenshot 2026-09-30 at 21 30 18" src="https://github.com/user-attachments/assets/bf025229-7632-4865-a7f7-e02e0790d361" />

필드명, 설명까지 모두 적혀있는 걸 볼 수 있었다. **listings(1): 등록된 모든 상품 조회, listings(2): 상품 ID를 써서 특정 상품 조회, myAccount: 로그인한 사용자 계명 및 잔여 크레딧 조회, promoCodes: 활성 프로모션 목록 조회.**
그런데 여기서 눈에 띄는 게 VENDOR ONLY. Discloses the active promo/discount table. Requires the vendor master key; rejected otherwise. << 라는 설명이다. 벤더 마스터키를 구해야 프로모션 코드를 구할 수 있을 것 같다. 


그렇다면, 저 키를 잘 구해서 프로모션 코드를 확인하고 그걸 쓰면 구매를 잘할 수 있지 않을까..? 이제 mutation 목록을 확인해보자.

<img width="1426" height="214" alt="Screenshot 2026-09-30 at 21 50 43" src="https://github.com/user-attachments/assets/c71358d0-7353-4e77-be7c-c32d74b02ae8" />


``
description
: 
"INTERNAL / MAINTENANCE. Re-syncs a vendor terminal and returns its diagnostics blob. Left enabled from the closed beta. TODO: gate before public launch."
name
: 
"vendorTerminalSync"`` 역시 제일 눈에 띄는 건 vendorTerminalSync다. 설명 문구를 보면 내부 운영자만 쓸 수 있는 기능인데 그게 지금 일반 사용자한테 그대로 보인다. TODO에 게이트를 만들어야 한다고 했지만, 아직 안 했나보다.


이제 벤더 인증 시 사용할 수 있는 핵심 필드를 찾아보자. 
<img width="1082" height="292" alt="Screenshot 2026-09-30 at 22 01 34" src="https://github.com/user-attachments/assets/3fc50b9a-109f-4068-9449-0fc72d26d30d" />

VendorDiagnostics 타입을 뜯어보면, vendorKey라는 완전 킥! 필드를 찾아낼 수 있다. NON_NULL로 되어있으니 비워둘 수 없는 문자열인 것도 확인 가능하다. 이 말인즉슨, 내가 저 값을 요청하면 키가 문자열 형태로 반환이 될 것이다. 



<img width="770" height="336" alt="Screenshot 2026-09-30 at 22 10 24" src="https://github.com/user-attachments/assets/76014099-b98e-4f64-80f1-a81af61ac908" />
vendorKey 값을 요청하니, 바로 키 값이 출력에 나타났다. **VND-MASTER-21d5f80206dffb6fa9ad5722** 이제 다음으로는 이 키를 저장하고, 프로모션 코드 목록을 쫙 가져오는 것이다. 여기서 사용할 수 있는 프로모션 코드를 얻을 수 있을 듯하다. 



<img width="1380" height="288" alt="Screenshot 2026-09-30 at 22 26 49" src="https://github.com/user-attachments/assets/f00d24d9-5adb-4ba3-8f2e-ba9e2f7d8ae9" />

정확하게 우리가 노리는 상품 번호 4042에 100프로 할인이 적용되는 코드 **FOUNDERS-100**이 보인다! 이제 이걸 입력하고 결제를 시도하면 될 것 같다. 

<img width="1048" height="860" alt="Screenshot 2026-09-30 at 22 29 44" src="https://github.com/user-attachments/assets/78309e21-ad95-4ae0-8b76-01c45a1ba5fc" />
<img width="1546" height="998" alt="Screenshot 2026-09-30 at 22 30 17" src="https://github.com/user-attachments/assets/e6fb17a7-4668-4c01-b276-36de2ae67c68" />

풀이 성공! GraphQL에 대해서는 처음 알아봐서 문제 풀이가 꽤 까다로웠던 것 같다. 


  












