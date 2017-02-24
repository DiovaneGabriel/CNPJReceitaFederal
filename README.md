# CNPJReceitaFederal
Codeigniter helper para consultar cadastros de pessoa jurídica na Receita Federal

Faça download do arquivo sample.zip se quiser testar na prática.
ou, mãos à obra.

Crie um helper com o nome "cnpj_receita_helper.php" e adicione o conteúdo abaixo.

```
<?php
//créditos à este cara tbm...
// http://blog.mayk.brito.net.br/php-curl-e-captcha-como-eu-entendi-e-aprendi-a-trabalhar-com-isso/
function get_captcha_cnpj_receita() {
	$url = "http://www.receita.fazenda.gov.br/pessoajuridica/cnpj/cnpjreva/captcha/gerarCaptcha.asp";
	$dir = "tmp" . DIRECTORY_SEPARATOR ;
	$cookie = FCPATH . $dir . 'receita.txt';
	if (! is_dir ( FCPATH . $dir )) {
		mkdir ( FCPATH . $dir, 0777, true );
		mkdir ( FCPATH . $dir, 0777, true );
	}
	$ch = curl_init ();
	curl_setopt_array ( $ch, array (
			CURLOPT_URL => $url,
			CURLOPT_COOKIEFILE => $cookie,
			CURLOPT_COOKIEJAR => $cookie,
			CURLOPT_FOLLOWLOCATION => 1,
			CURLOPT_RETURNTRANSFER => 1,
			CURLOPT_BINARYTRANSFER => 1,
			CURLOPT_HEADER => 0 
	) );
	
	$data = curl_exec ( $ch );
	curl_close ( $ch );
	
	$arquivo = $dir ."receita.gif";
	$fp = fopen ( $arquivo, 'w' );
	fwrite ( $fp, $data );
	fclose ( $fp );
	
	return $arquivo;
}
function get_cnpj_receita($cnpj, $letras) {
	$reffer = "http://google.com";
	$agent = "Mozilla/5.0 (Windows; U; Windows NT 5.0; en-US; rv:1.4) Gecko/20030624 Netscape/7.1 (ax)";
	$url = "http://www.receita.fazenda.gov.br/pessoajuridica/cnpj/cnpjreva/valida.asp";
	$post_fields = "origem=comprovante&cnpj=".$cnpj."&txtTexto_captcha_serpro_gov_br=".$letras."&idSom=&submit1=Consultar&search_type=cnpj";
	$dir = "tmp" . DIRECTORY_SEPARATOR ;
	$cookie = FCPATH . $dir . 'receita.txt';
	$ch = curl_init ();
	
	curl_setopt_array ( $ch, array (
			CURLOPT_URL => $url,
			CURLOPT_POST => 1,
			CURLOPT_POSTFIELDS => $post_fields,
			CURLOPT_USERAGENT => $agent,
			CURLOPT_REFERER => $reffer,
			CURLOPT_COOKIEFILE => $cookie,
			CURLOPT_COOKIEJAR => $cookie,
			CURLOPT_FOLLOWLOCATION => 1,
			CURLOPT_RETURNTRANSFER => 1,
			CURLOPT_HEADER => 0 
	) );
	
	$result = curl_exec ( $ch );
	
	curl_close ( $ch );
	
	if (preg_match ( "/<title>Emiss.*<\/title>/siU", $result )) {
		return false;
	}
	
	$html = utf8_encode ( $result );
	
	$inicio = strrpos ( $html, '<!-- Início Linha NÚMERO DE INSCRIÇÃO -->' );
	$fim = strrpos ( $html, '<!-- Fim Linha NÚMERO DE INSCRIÇÃO -->' );
	$numero_inscricao = substr ( $html, $inicio, $fim - $inicio );
	$cnpj = explode ( '<b>', $numero_inscricao )[1];
	$cnpj = substr ( $cnpj, 0, strrpos ( $cnpj, '</b>' ) );
	
	$inicio = strrpos ( $html, '<!-- Início Linha NOME EMPRESARIAL -->' );
	$fim = strrpos ( $html, '<!-- Fim Linha NOME EMPRESARIAL -->' );
	$nome_empresarial = substr ( $html, $inicio, $fim - $inicio );
	$razao_social = explode ( '<b>', $nome_empresarial )[1];
	$razao_social = substr ( $razao_social, 0, strrpos ( $razao_social, '</b>' ) );
	
	$inicio = strrpos ( $html, '<!-- Início Linha LOGRADOURO -->' );
	$fim = strrpos ( $html, '<!-- Fim Linha LOGRADOURO -->' );
	$endereco = substr ( $html, $inicio, $fim - $inicio );
	$logradouro = explode ( '<b>', $endereco )[1];
	$logradouro = substr ( $logradouro, 0, strrpos ( $logradouro, '</b>' ) );
	$numero = explode ( '<b>', $endereco )[2];
	$numero = substr ( $numero, 0, strrpos ( $numero, '</b>' ) );
	$complemento = explode ( '<b>', $endereco )[3];
	$complemento = substr ( $complemento, 0, strrpos ( $complemento, '</b>' ) );
	
	$inicio = strrpos ( $html, '<!-- Início Linha CEP -->' );
	$fim = strrpos ( $html, '<!-- Fim Linha CEP -->' );
	$endereco2 = substr ( $html, $inicio, $fim - $inicio );
	$cep = explode ( '<b>', $endereco2 )[1];
	$cep = substr ( $cep, 0, strrpos ( $cep, '</b>' ) );
	$bairro = explode ( '<b>', $endereco2 )[2];
	$bairro = substr ( $bairro, 0, strrpos ( $bairro, '</b>' ) );
	$cidade = explode ( '<b>', $endereco2 )[3];
	$cidade = substr ( $cidade, 0, strrpos ( $cidade, '</b>' ) );
	$uf = explode ( '<b>', $endereco2 )[4];
	$uf = substr ( $uf, 0, strrpos ( $uf, '</b>' ) );
	
	$inicio = strrpos ( $html, '<!-- Início Linha SITUAÇÃO CADASTRAL -->' );
	$fim = strrpos ( $html, '<!-- Fim Linha SITUACAO CADASTRAL -->' );
	$situacao_cadastral = substr ( $html, $inicio, $fim - $inicio );
	$situacao_cadastral = explode ( '<b>', $situacao_cadastral )[1];
	$situacao_cadastral = substr ( $situacao_cadastral, 0, strrpos ( $situacao_cadastral, '</b>' ) );
	
	$clear_number = function ($string) {
		$string = preg_replace ( "/[^0-9,.]/", "", $string );
		$string = str_replace ( '.', '', $string );
		$string = str_replace ( ',', '', $string );
		return $string;
	};
	
	$empresa = [ ];
	$empresa ["cnpj"] = $clear_number ( $cnpj );
	$empresa ["razao_social"] = trim ( $razao_social );
	$empresa ["logradouro"] = trim ( $logradouro );
	$empresa ["numero"] = trim ( $numero );
	$empresa ["complemento"] = trim ( $complemento );
	$empresa ["cep"] = $clear_number ( $cep );
	$empresa ["bairro"] = trim ( $bairro );
	$empresa ["cidade"] = trim ( $cidade );
	$empresa ["uf"] = trim ( $uf );
	$empresa ["situacao_cadastral"] = trim ( $situacao_cadastral );
	$empresa = ( object ) $empresa;
	
	return $empresa;
}
```

Adicione os seguistes helpers no autoload

```
$autoload['helper'] = array('url','file');
```

Crie um controlle mais ou menos tipo esse

```
<?php
defined ( 'BASEPATH' ) or exit ( 'No direct script access allowed' );
class Consulta_cnpj_receita extends CI_Controller {
	function __construct() {
		parent::__construct ();
		$this->load->helper ( 'cnpj_receita_helper' );
	}
	public function index() {
		$data ['captcha'] = get_captcha_cnpj_receita ();
		$this->load->view ( 'consulta_cnpj_receita', $data );
	}
	public function consulta_cnpj() {
		$cnpj = $_POST ['cnpj'];
		$captcha = $_POST ['captcha'];
		
		$return = get_cnpj_receita ( $cnpj, $captcha );
		if (! $return) {
			redirect ( base_url ( "consulta_cnpj_receita" ) );
		}
		var_dump ( $return );
	}
}
```

e uma view mais ou menos como essa

```
<img src='<?php echo base_url($captcha)?>' />
<form action="<?php echo base_url('consulta_cnpj_receita/consulta_cnpj')?>" method="POST">
	captcha <input size="8" maxlength="6" name="captcha"> cnpj <input
		size="16" maxlength="14" name="cnpj"> <input type="submit">
</form>
```

e pronto, pra testar acesse "sua_url_base"/consulta_cnpj_receita (ex.: http://localhost/sample/consulta_cnpj_receita)

