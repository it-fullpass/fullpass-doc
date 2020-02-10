# Implementando o login Fullpass

O login single sign on fullpass é baseado no modelo OAuth2. O processo segue aspráticas comuns do fluxo OAuh2/OpenID.

Neste documento tem-se uma explicação desse processo e sua integração com a APIfullpass para checagem de perfil e track de pageview.

Para fazer login de algum usuário, é preciso requisitar um authentication code no fullpass. Esse code deve ser trocado por um token e só pode ser usado uma vez. Com esse token, você site parceiro pode acessar a API fullpass e confirmar que usuário ainda temassinatura válida.

A seguir passos de como implementar.

## 1o Passo - login usuário

Acessar a página de login fullpass. Para isso você deve acessar um URL no seguinte formato:

<code>https://kc.fullpass.me/auth/realms/fullpass/protocol/openid-connect/auth?clie</code>

CLIENT_ID = Id configurado do seu site URL_RETORNO = URL para onde será redirecionado um login bem sucedido. Deve ser URL encoded.

Parâmetros de segurança opcionais: UUID_STATE = um UUID que pode ser gerado no seu servidor para validar que a resposta veio do nosso servidor UUID_NONCE = um UUID que pode ser gerado no seu servidor e pode ser validado se o token for decodificado

Caso seja bem sucedido, vai receber uma chamada de retorno com o código e o mesmo valor de state enviado
