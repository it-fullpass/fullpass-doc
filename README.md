
# Implementando o login Fullpass

O login single sign on fullpass é baseado no modelo OAuth2. O processo segue as práticas comuns do fluxo OAuh2/OpenID.

Neste documento tem-se uma explicação desse processo e sua integração com a API fullpass para checagem de perfil e track de pageview.

Para fazer login de algum usuário, é preciso requisitar um `authentication code` no full pass. Esse code deve ser trocado por um token e só pode ser usado uma vez.
Com esse token, você site parceiro pode acessar a API fullpass e confirmar que usuário ainda tem assinatura válida.  

A seguir passos de como implementar.

### 1o Passo - login usuário

Acessar a página de login fullpass. Para isso você deve acessar um URL no seguinte formato: 

<code>https://kc.fullpass.me/auth/realms/fullpass/protocol/openid-connect/auth?client_id=[CLIENT_ID]&redirect_uri=[URL_RETORNO]&state=[UUID_STATE]&response_mode=query&response_type=code&scope=openid&nonce=[UUID_NONCE]</code>

- `CLIENT_ID` = Id configurado do seu site. Você deve combinar esse valor com a equipe fullpass ao integrar o seu site.
- `URL_RETORNO` = URL para onde será redirecionado um login bem sucedido. Deve ser URL encoded.


Parâmetros de segurança opcionais: 
`UUID_STATE` = um UUID que pode ser gerado no seu servidor para validar que a resposta veio do nosso servidor

`UUID_NONCE` = um UUID que pode ser gerado no seu servidor e pode ser validado se o token for decodificado

Caso seja bem sucedido, vai receber uma chamada de retorno com o código e o mesmo valor de state enviado

    [URL_SITE_PARCEIRO]/?state=[UUID_STATE]&session_state=[UUID_SESSION_STATE]&code=[AUTH_CODE]

- `UUID_STATE` = o mesmo UUID que foi enviado no começo do login
- `AUTH_CODE` = código de autenticação a ser usado para obter um token
- `UUID_SESSION_STATE` = um ID da sessão, que será reconfirmado ao requisitar o TOKEN (passo 2)


### 2o Passo - obtenção de token

Esa chamada deve ser feita no *back end* da sua aplicação.
Como fazer um request de token:

Ex. usando node JS:
<pre>
    var request = require("request");
    
    var options = { method: 'POST',
      url: 'https://kc.fullpass.me/auth/realms/fullpass/protocol/openid-connect/token',
      headers: 
       { 
         Accept: 'application/json',
         'Content-Type': 'application/x-www-form-urlencoded' },
      form: 
       {
       code: '[AUTH_CODE]'.
       grant_type: 'authorization_code',
       client_id: '[CLIENT_ID]',
       redirect_uri: '[REDIRECT_URI]' 
       } 
    };
    
    request(options, function (error, response, body) {
      if (error) throw new Error(error);
    
      console.log(body);
    });
</pre>

Exemplo de resposta de token:

<pre>
    {
        "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJvNXdXa3ZHdE9uNXNzYVgxQ0VTRlNsSmdfZkZBeTBGRFJydUloaFViTC1jIn0.eyJqdGkiOiIwZTRlNzg2MS1mMTQyLTQ0YTAtOWQ4OS1kZDA2OWUzNzU5NjciLCJleHAiOjE1Njg2MDI5OTIsIm5iZiI6MCwiaWF0IjoxNTY4NjAyNjkyLCJpc3MiOiJodHRwOi8vc2VjdXJlLmZ1bGxwYXNzLmNvbS5icjo4MDgwL2F1dGgvcmVhbG1zL2Z1bGxwYXNzIiwiYXVkIjoiYWNjb3VudCIsInN1YiI6IjVmZjZiMzI4LWRhYWItNGU1YS05YzcyLTc3Zjc4NWRhMTkxOSIsInR5cCI6IkJlYXJlciIsImF6cCI6InBsYW5ldGEtZGlhcmlvIiwibm9uY2UiOiJhZDQ5NDE4OC1mNWQ3LTRlM2YtOWVlNC1iNTZjNDRhMWM3NDAiLCJhdXRoX3RpbWUiOjE1Njg2MDI2MDAsInNlc3Npb25fc3RhdGUiOiJmMzk2YWI4MS03NWRlLTRmN2EtODk0OS1mODNhMzhkN2FhOGMiLCJhY3IiOiIwIiwiYWxsb3dlZC1vcmlnaW5zIjpbImh0dHA6Ly93d3cucGxhbmV0YWRpYXJpby5jb20uYnI6ODA4MiJdLCJyZWFsbV9hY2Nlc3MiOnsicm9sZXMiOlsib2ZmbGluZV9hY2Nlc3MiLCJ1bWFfYXV0aG9yaXphdGlvbiJdfSwicmVzb3VyY2VfYWNjZXNzIjp7ImFjY291bnQiOnsicm9sZXMiOlsibWFuYWdlLWFjY291bnQiLCJtYW5hZ2UtYWNjb3VudC1saW5rcyIsInZpZXctcHJvZmlsZSJdfX0sInNjb3BlIjoib3BlbmlkIHByb2ZpbGUgZW1haWwiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsInByZWZlcnJlZF91c2VybmFtZSI6InRlc3QifQ.VOvclk0qgKs4v9J6tQAe--N3WZq1AkfqgNAK89s_gusEx4RwywkNljnzZaZ7Zb7etMiguw88edwT-KOPwCnwxYzZRXKH8UljTgizvwrPDGpxU9YlmyvLYgSTVEz27DBaBqOppS08Qo2Vm1Fy-kGxd6dUFMmPXD-BRvwPz5JUAiQ4-_AxIVgGWJDp-0wXuacDa10TXybAR_KKeSnupkAfuBOR2HeN09XyTSOjDcbKE_lhzI8jUYZoibBxmWKLtYv7n17Kb1IC6N3bpSTe0D7HLgg6lw5a0KnJkTnUHUXexFg3TA6aCsScNkCLDh9ikhMpM88WdUSO2N_ZU6NaVL2TKA",
        "expires_in": 300,
        "refresh_expires_in": 1800,
        "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJkMDc2NGIyMy01ZTE4LTQwNDItYTA0My1mNzFjOGI0ZDY4NzUifQ.eyJqdGkiOiI5ZmQ1ODkwNS1jYjhhLTQ2NTQtYjMzMS1hYWJmMTUwM2NlZmYiLCJleHAiOjE1Njg2MDQ0OTIsIm5iZiI6MCwiaWF0IjoxNTY4NjAyNjkyLCJpc3MiOiJodHRwOi8vc2VjdXJlLmZ1bGxwYXNzLmNvbS5icjo4MDgwL2F1dGgvcmVhbG1zL2Z1bGxwYXNzIiwiYXVkIjoiaHR0cDovL3NlY3VyZS5mdWxscGFzcy5jb20uYnI6ODA4MC9hdXRoL3JlYWxtcy9mdWxscGFzcyIsInN1YiI6IjVmZjZiMzI4LWRhYWItNGU1YS05YzcyLTc3Zjc4NWRhMTkxOSIsInR5cCI6IlJlZnJlc2giLCJhenAiOiJwbGFuZXRhLWRpYXJpbyIsIm5vbmNlIjoiYWQ0OTQxODgtZjVkNy00ZTNmLTllZTQtYjU2YzQ0YTFjNzQwIiwiYXV0aF90aW1lIjowLCJzZXNzaW9uX3N0YXRlIjoiZjM5NmFiODEtNzVkZS00ZjdhLTg5NDktZjgzYTM4ZDdhYThjIiwicmVhbG1fYWNjZXNzIjp7InJvbGVzIjpbIm9mZmxpbmVfYWNjZXNzIiwidW1hX2F1dGhvcml6YXRpb24iXX0sInJlc291cmNlX2FjY2VzcyI6eyJhY2NvdW50Ijp7InJvbGVzIjpbIm1hbmFnZS1hY2NvdW50IiwibWFuYWdlLWFjY291bnQtbGlua3MiLCJ2aWV3LXByb2ZpbGUiXX19LCJzY29wZSI6Im9wZW5pZCBwcm9maWxlIGVtYWlsIn0.kPuU95TGCJeHKMiF5GqJkDya4gxpywT6ER-0hDMZlnE",
        "token_type": "bearer",
        "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJvNXdXa3ZHdE9uNXNzYVgxQ0VTRlNsSmdfZkZBeTBGRFJydUloaFViTC1jIn0.eyJqdGkiOiJmZGM0ODhmMi03NTc5LTQ2YWUtODdmMy05NjQ1ODhmY2VmOWMiLCJleHAiOjE1Njg2MDI5OTIsIm5iZiI6MCwiaWF0IjoxNTY4NjAyNjkyLCJpc3MiOiJodHRwOi8vc2VjdXJlLmZ1bGxwYXNzLmNvbS5icjo4MDgwL2F1dGgvcmVhbG1zL2Z1bGxwYXNzIiwiYXVkIjoicGxhbmV0YS1kaWFyaW8iLCJzdWIiOiI1ZmY2YjMyOC1kYWFiLTRlNWEtOWM3Mi03N2Y3ODVkYTE5MTkiLCJ0eXAiOiJJRCIsImF6cCI6InBsYW5ldGEtZGlhcmlvIiwibm9uY2UiOiJhZDQ5NDE4OC1mNWQ3LTRlM2YtOWVlNC1iNTZjNDRhMWM3NDAiLCJhdXRoX3RpbWUiOjE1Njg2MDI2MDAsInNlc3Npb25fc3RhdGUiOiJmMzk2YWI4MS03NWRlLTRmN2EtODk0OS1mODNhMzhkN2FhOGMiLCJhY3IiOiIwIiwiZW1haWxfdmVyaWZpZWQiOmZhbHNlLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJ0ZXN0In0.HMSnf5XWNexfb3TkvgnSvi785scUjLw46spbdl3JLcIl1r_rXJFGJTAVsWjJ28AKE-E7HAEI56OCWZ5iVdfcjPm_gX50wij_O2MMTYUbEOGeZEunb1s9LU0_43B4ya7vdZ5wFDNJ57hA0vsiUSzuwdxg5uacaX7ajlzBOvy209xV3j147l6Ef_1VIQwJ6zJ_Nb3sCkFN1w9lqKKN5Oq8BdNpa-85yAJaRUuNkqdWD-7Td6UQKTHlO1uhH1ik83IiLnXKJoP3NPFUTwvRc8aGhfDptGGehf_k7wNjtSQwuVIDiSrkdd_vaiUyVqzw4AjaUrv9QMjbg2FBZlbvBmyN_Q",
        "not-before-policy": 0,
        "session_state": "f396ab81-75de-4f7a-8949-f83a38d7aa8c",
        "scope": "openid"
    }
</pre>

O `session_state` deve bater com o `[UUID_SESSION_STATE]` recebido.

Importante notar:
- `access_token` - o token de acesso que deve ser usado para chamar a API do fullpass pra validar o usuário    
- `expires_in` - tempo em segundos em que o token de acesso expira
- `refresh_token` - o novo refresh token a ser usado

### 3o Passo - verificar assinatura

No seu backend, chamar a API do FULLPASS para verificar se o perfil ainda tem acesso/assinatura válida.

Ex de chamada de verificação de assinatura:

Deve se fazer um request HTTP usando o token recuperado anteriormente

GET:

<code> https://partner-api.fullpass.me/partner-api/profile</code>

Exemplo de resposta:

<code> {"subscriptionValid":true} </code>
    
Exemplo NodeJS:

<pre>
    var request = require("request");
    
    var options = { method: 'GET',
      url: 'https://partner-api.fullpass.me/partner-api/profile',
      headers: 
       { 'cache-control': 'no-cache',
         Connection: 'keep-alive',
         'Accept-Encoding': 'gzip, deflate',
         Host: 'partner-api.fullpass.me',
         'Cache-Control': 'no-cache',
         Authorization: 'Bearer [TOKEN]',
         Accept: 'application/json',
         'Content-Type': 'application/x-www-form-urlencoded' } };
    
    request(options, function (error, response, body) {
      if (error) throw new Error(error);
    
      console.log(body);
    });
</pre>

Onde `[TOKEN]` é o token de autenticação obtido.     

Feito isso, seu site pode criar a sua própria sessão logada.

De tempos em tempos, pode ser necessário chamar novamente a API fullpass para garantir que a sessão/assinatura do usuário ainda é válida.

Caso o token esteja expirado, um código http `401` será retornado.

Além disso, é necessário fazer o processo de `refresh token` para que o usuário não precise fazer login novamente no seu site.

Para saber o momento correto de fazer o refresh, deve-se observar o seu tempo de expiração (`expires_in` na resposta do token). 

Por favor não fazer refesh token desnecessariamente para não criar uma carga desnecessária em nossos servidores.

#### Como fazer um request de refresh token

Usando node JS:

<pre>
    var request = require("request");

    var options = { method: 'POST',
      url: 'https://kc.fullpass.me/auth/realms/fullpass/protocol/openid-connect/token',
      headers: 
       { 
         Accept: 'application/json',
         'Content-Type': 'application/x-www-form-urlencoded' },
      form: 
       { refresh_token: '[TOKEN]',
         client_id: '[CLIENT_ID]',
         grant_type: 'refresh_token' } };
    
    request(options, function (error, response, body) {
      if (error) throw new Error(error);
    
      console.log(body);
    });
</pre>

Exemplo de resposta de token refresh é equivalente a chamada de login inicial.

## Track page view

Para fazer o track de page view deve-se usar a API no browser do usuário já depois de validado o login nos passos anteriores.
O mesmo token deve ser passado para a renderização da sua página, e o parâmetro é usado então pelo javascript do fullpass 

Ex de implementação:
<pre>
    <script>
    var fullpassClientId = '[CLIENT_ID_PARCEIRO]'; // o client id utilizado para fazer login
    var fullpassToken = '<%= fullpassToken %>'; //variável preenchida pelo backend do parceiro
    var freeView = false; //deve ser passado para indicar se a página é gratuita ou não
    var loginCallBackUrl = '[CALLBACK URL]'; //opcional para autologin
    </script>
    <script src="https://partner-api.fullpass.me/partner-api/js/fullpass.js" type="text/javascript"></script>
</pre>

O valor de `fullpassClientId` deve ser o mesmo utilizado para a rotina de login no seu backend.

O script do fullpass espera que exista uma variável `fullpassToken`, que é usada para contabilizar o acesso na página sendo visualizada no momento.
Ou seja, se o usuário tiver sido logado pelo seu back end, esse valor deve ser passado.

O parâmetro `freeView` indica se a página sendo exibida deve ser contabilizada ou não. 
Por exemplo, algumas matérias no seu site podem ser gratuitas e não fazer sentido contar o page view. 

Além disso, é desejável passar a variável opcional `loginCallBackUrl` para autologin. 
Esta feature deve ser implementada no futuro, e se estiver disponível, será usada para fazer login automático para usuários que já estejam logados no fullpass, mesmo que em outro site que não o do parceiro.


exemplo de implementação:

<pre>
    <script>
    
    <%if (fullpassToken) {%>
        var fullpassToken = '<%= fullpassToken %>';
        var freeView = true; //na home, por exemplo
    <%} else {%>
        var freeView = true; //usuario nao logado, mas parceiro deseja fazer o track free
    <%}%>
    
        var loginCallBackUrl = 'http://www.jornalsensacional.com.br/loginfullpass';
        var fullpassClientId = 'jornal-sensacional';
    </script>
    <script src="https://partner-api.fullpass.me/partner-api/js/fullpass.js" type="text/javascript"></script>
    
  </pre>
