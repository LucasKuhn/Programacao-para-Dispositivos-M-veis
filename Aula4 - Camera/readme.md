# Camera

### 0. Permissões

É necessário ter permissão para ler o storage. Vamos adicionar a permissão também para escrever, e a feature de acessar a câmera:

```xml
<!-- src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-feature android:name="android.hardware.camera" android:required="true" />

```

Adicionar as permissões no Manifest não é o suficiente para todas versões do Android, por isso adicionamos no método onCreate algumas checagens para pedir as permissões:

```java
// Pede permissão para acessar as mídias gravadas no dispositivo

if (ContextCompat.checkSelfPermission(this,
        Manifest.permission.READ_EXTERNAL_STORAGE)
        != PackageManager.PERMISSION_GRANTED) {
    if (ActivityCompat.shouldShowRequestPermissionRationale(this,
            Manifest.permission.READ_EXTERNAL_STORAGE)) {
    } else {
        ActivityCompat.requestPermissions(this,
                new String[]{Manifest.permission.READ_EXTERNAL_STORAGE},
                PERMISSAO_REQUEST);
    }
}
// Pede permissão para escrever arquivos no dispositivo
if (ContextCompat.checkSelfPermission(this,
        Manifest.permission.WRITE_EXTERNAL_STORAGE)
        != PackageManager.PERMISSION_GRANTED) {
    if (ActivityCompat.shouldShowRequestPermissionRationale(this,
            Manifest.permission.WRITE_EXTERNAL_STORAGE)) {
    } else {
        ActivityCompat.requestPermissions(this,
                new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},
                PERMISSAO_REQUEST);
    }
}
```
E adicionamos um inteiro para o PERSMISSAO_REQUEST
```java
public class MainActivity extends AppCompatActivity {
  // ...
  private final int PERMISSAO_REQUEST = 2;
}
```

### 1. Preparar o Layout para a imagem e os botões

Criar uma basic activity
Trocar o ConstraintLayout por um LinearLayout com GridLayout (nesse caso, 3 colunas e 8 linhas)
```xml
<!-- layout/activity_main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:app="http://schemas.android.com/apk/res-auto"
 xmlns:tools="http://schemas.android.com/tools"
 android:layout_width="match_parent"
 android:layout_height="match_parent"
 android:gravity="center"
 android:layout_weight="0"
 android:orientation="vertical">
 <GridLayout xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:app="http://schemas.android.com/apk/res-auto"
 xmlns:tools="http://schemas.android.com/tools"
 android:id="@+id/GridLayout1"
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:columnCount="3"
 android:orientation="horizontal"
 android:rowCount="8"
 android:useDefaultMargins="true"
 tools:context=".MainActivity">

 <!--Aqui vai a imagem -->
 <LinearLayout
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:layout_columnSpan="3">

 <!-- Aqui vão os botões -->
 </LinearLayout>
 </GridLayout>
</LinearLayout>
```
- `match_parent` Serve para o LinearLayout ser do tamanho do parent (no caso, do tamanho da tela inteira)

### 2. Adicionar um PhotoView

O ImageView tem poucas funcionalidades, por isso vamos utilizar o PhotoView que permite zoom, abrir com 2 cliques, etc. Ele herda de ImageView, por isso podemos declará-lo como `private ImageView imagem;`

  - https://github.com/chrisbanes/PhotoView

O Gradle é quem gerencia as dependências. Adicionar no build.gradle (conforme descrito no Github - Usar versão 2.1.4)

```xml
<!-- layout/activity_main.xml -->
<com.github.chrisbanes.photoview.PhotoView
    android:id="@+id/pv_image"
    android:layout_width="324dp"
    android:layout_height="439dp"
    android:layout_rowSpan="7"
    android:layout_columnSpan="3"
    android:layout_gravity="fill_horizontal"
    android:background="@color/colorPrimary"
    android:scaleType="centerInside" />
```
- `439dp` Definir o tamanho em SP Scale-independent Pixels ou em DP Density-Independent Pixels

Declarar uma variável para salvar o ImageView, e obter a referência dele dentro do onCreate:

```java
public class MainActivity extends AppCompatActivity {
  // ...
  private ImageView imagem;

  protected void onCreate(Bundle savedInstanceState) {
    // ...
    imagem = findViewById(R.id.pv_image);
  }
}
```
- `pv_image`: É o id que nós definimos para o PhotoView

### 3. Fazer o método mostraFoto
Para mostrar uma foto selecionada na PhotoView
```java
private void mostraFoto(String caminho) {
    Bitmap bitmap = BitmapFactory.decodeFile(caminho);
    imagem.setImageBitmap(bitmap);
}
```
- Este método é chamado quando volta a Intent de abrir a câmera e quando volta a Intent de abrir a galeria de imagens.
- `imagem` é o a variavel que definimos no começo da MainActivity, e pegamos a referencia no método onCreate

### 4. Adicionar os botões no Layout
```xml
<Button
    android:id="@+id/btnCarrega"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="fill_horizontal"
    android:onClick="buscar"
    android:text="Carrega Imagem" />

<Button
    android:id="@+id/btnFoto"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="fill_horizontal"
    android:onClick="tirarFoto"
    android:text="Tira Foto" />

<Button
    android:id="@+id/btnCompartilha"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="fill_horizontal"
    android:onClick="compartilhar"
    android:text="Compartilhar Foto" />
```
- Observe que os métodos onClick já estão setados. Vamos criar estes métodos em seguida
### 4. Adicionar método de abrir imagem

O botão invoca uma activity que vai e volta (vai para a galeria, e volta com o resultado), então devemos fazer um StartActivityforResult

```java
// java/<your-project>/MainActivity.java

private final int GALERIA_IMAGENS = 1;

public void buscar(View view) {

    Intent intent = new Intent(Intent.ACTION_PICK,
            android.provider.MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
    startActivityForResult(intent, GALERIA_IMAGENS);
}

```
- `ACTION_PICK`: É uma intent de selecionar algum arquivo no dispositivo
- `EXTERNAL_CONTENT_URI`: Path para o arquivo
- `GALERIA_IMAGENS`: É um inteiro envido para identificar a ação na hora de voltar


Código para gerenciar o Activity Result
```java

private File arquivoFoto = null;

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (resultCode == RESULT_OK && requestCode == GALERIA_IMAGENS) {
        Uri selectedImage = data.getData();
        String[] filePath = {MediaStore.Images.Media.DATA};
        Cursor c = getContentResolver().query(selectedImage, filePath, null,
                null, null);
        c.moveToFirst();
        int columnIndex = c.getColumnIndex(filePath[0]);
        String picturePath = c.getString(columnIndex);
        c.close();
        arquivoFoto = new File(picturePath);
        mostraFoto(arquivoFoto.getAbsolutePath());
    }
}
```
- `Intent data`: É o intent que está voltando, ele contém o caminho para a imagem
- `Uri selectedImage`: Caminho para a imagem
- `filePath`: Aonde estão as imagens no dispositivo
- `getContentResolver`: Gerenciador de conteúdo (content provider)
- `picturePath`: Este sim, é o caminho completo para a imagem

### 5. Adicionar o provider path
Cada software vai ter uma pasta aonde ele está instalado e onde ficam os seus arquivos.
Este providers_path diz para gravar na própria pasta aonde está rodando o dispositivo: `.`
Criar o arquivo providers_path.xml dentro de uma pasta chamada xml:
```xml
<!-- main/res/xml/providers_path.xml -->
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="external_files" path="."/>
</paths>
```
Este arquivo então deve ser referenciado no AndroidManifest
```xml
<provider
  android:name="androidx.core.content.FileProvider"
  android:authorities="${applicationId}.provider"
  android:exported="false"
  android:grantUriPermissions="true">
  <meta-data
    android:name="android.support.FILE_PROVIDER_PATHS"
    android:resource="@xml/providers_path"/>
</provider>
```

### 6. Fazer o método tirarFoto

```java
// MainActivity.java

private final int CAMERA = 3;

public void tirarFoto(View view) {
    Intent takePictureIntent = new
            Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        try {
            arquivoFoto = criaArquivo();
        } catch (IOException ex) {
            mostraAlerta(getString(R.string.erro), getString(R.string.erro_salvando_foto));
        }
        if (arquivoFoto != null) {
            Uri photoURI = FileProvider.getUriForFile(getBaseContext(),
                    getBaseContext().getApplicationContext().getPackageName() +
                            ".provider", arquivoFoto);
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI);
            startActivityForResult(takePictureIntent, CAMERA);
        }
    }
}

private File criaArquivo() throws IOException {
    String timeStamp = new
            SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
    File pasta = Environment.getExternalStoragePublicDirectory(
            Environment.DIRECTORY_PICTURES);
    File imagem = new File(pasta.getPath() + File.separator
            + "JPG_" + timeStamp + ".jpg");
    return imagem;
}
```
- O método `mostraAlerta` está definido no final deste tutorial

E adicionar no onActivityResult um if para lidar com o resultado da CAMERA:

```java
if (resultCode == RESULT_OK && requestCode == CAMERA) {
    sendBroadcast(new Intent(
            Intent.ACTION_MEDIA_SCANNER_SCAN_FILE,
            Uri.fromFile(arquivoFoto))
    );
    mostraFoto(arquivoFoto.getAbsolutePath());
}
```
- `sendBroadcast`: Diz para a camera salvar a foto tirada como arquivo

### 7. Fazer a ação de compartilhar

```java
public void compartilhar(View view) {
    Uri uri = null;
    if(arquivoFoto!=null) {
        uri = FileProvider.getUriForFile(getBaseContext(),
                getBaseContext().getApplicationContext().getPackageName() +
                        ".provider", arquivoFoto);
        if(uri!=null) {
            compartilharImagem(uri,"image/jpg");
        }
    }
}
```
- `arquivoFoto`: É uma variável do tipo File aonde a foto foi salva depois de selecionada

```java
private void compartilharImagem(Uri uri, String tipo){
    Intent intent = new Intent(android.content.Intent.ACTION_SEND);
    intent.putExtra(Intent.EXTRA_STREAM, uri);
    intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
    intent.setType(tipo);
    startActivity(intent);
}
```
- `ACTION_SEND`: Intenção de enviar
- `putExtra`: Enviar o caminho da imagem para o aplicativo que de fato vai fazer o comparilhamento


## DICA: AlertDialog
```java
private void mostraAlerta(String titulo, String mensagem) {
    AlertDialog alertDialog = new AlertDialog.Builder(MainActivity.this).create();
    alertDialog.setTitle(titulo);
    alertDialog.setMessage(mensagem);
    alertDialog.setButton(AlertDialog.BUTTON_NEUTRAL, getString(R.string.ok),
            new DialogInterface.OnClickListener() {
                public void onClick(DialogInterface dialog, int which) {
                    dialog.dismiss();
                }
            });
    alertDialog.show();
}
```

# Referencias

Layouts
- https://imasters.com.br/android/como-usar-o-constraint-layout-no-android
- https://medium.com/ci-t-mobile/constraintlayout-a-evolu%C3%A7%C3%A3o-do-relativelayout-6b8ade707f22
- https://www.androidpro.com.br/blog/desenvolvimento-android/android-layouts-viewgroups-intro/

Editor de Layout
- https://developer.android.com/studio/write/layout-editor.html

Intents Disponíveis
- https://developer.android.com/guide/components/intents-common
- https://www.tutorialspoint.com/android/android_intent_standard_actions.htm

Providers
- https://developer.android.com/reference/android/provider/package-summary.html
