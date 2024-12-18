# Unity-CamaraMicro
- Por Pablo Santana González

Adjunto el proyecto entero, como paquete, para poder probarlo ya que reproducir los audios, una parte central de la práctica, es algo que resulta imposible mediante una imagen o un gif.

La escena `Ej1` representa el ejercicio 1, mientras que la escena `Ej2` representa el resto de ejercicios.

A lo largo de la descripción de la práctica se describirán los controles que se utilizan, pero en el caso de la escena 2 no hace falta memorizarlos, he incluido un canvas con unas anotaciones explicando los controles brevemente.

## Escena 1.
- Monté una escena sencilla con una araña y un huevo. Cuando se pulsan las flechas izquierda y derecha se desplaza a la araña por la pantalla. Cuando el huevo y la araña colisionan, se reproduce un audio utilizando un objeto `AudioSource` que se ha agregado como componente a la araña. Desde el inspector, se le puede cambiar el clip de audio (`AudioClip`) utilizado para el sonido mirando la componente de `AudioSource`.

- El siguiente es el código desarrollado:
```cs
public class TowardsTheEgg : MonoBehaviour
{
    public float speed = 2f;
    private AudioSource vfx;
    // Start is called before the first frame update
    void Start()
    {
        vfx = GetComponent<AudioSource>();
    }

    // Update is called once per frame
    void Update()
    {
        if (Input.GetKey(KeyCode.RightArrow)) {
            this.transform.Translate(this.transform.forward * Time.deltaTime * speed, Space.World);
        } else if (Input.GetKey(KeyCode.LeftArrow)) {
            this.transform.Translate(-this.transform.forward * Time.deltaTime * speed, Space.World);
        }
    }

    void OnCollisionEnter(Collision other) {
        if (other.gameObject.tag == "egg") {
            Debug.Log("The spider touched the egg!");
            vfx.Play(); // Reproducir el sonido asignado en la componente.
        }
    }
}
```

 ## Escena 2
 - La siguiente escena consta de una TV conseguida a partir de un [asset de Unity](https://assetstore.unity.com/packages/3d/props/electronics/tv-set-26193). Me encontré con un problema al usarlo, ya que al intentar poner la `WebCamTexture` al material de la pantalla, la imagen se encontraba invertida. Como solución, cree un plano que puse delante de la pantalla para poder rotarlo a conveniencia.

La lógica de la escena se divide en dos partes: audio y vídeo.
- La de audio comienza a grabar según se comienza la escena. Hay un bool encargado de determinar si se está grabando o reproduciendo y se usan dos botones diferentes. El motivo de esto es que a veces al pulsar el botón tomaba la entrada varias veces. La R se usa para reproducir el audio y la L se usa para grabarlo. Si se está grabando no se puede empezar a grabar de nuevo, hasta que se reproduzca el audio.
```cs
public class Recorder : MonoBehaviour
{
    public int maxClipSize = 300;
    private AudioClip currentAudio;
    private AudioSource src;
    private bool recording = true;
    // Start is called before the first frame update
    void Start()
    {
        currentAudio = Microphone.Start("", false, maxClipSize, 44100);
        src = GetComponent<AudioSource>();
        Debug.Log("Recording...");
    }

    // Update is called once per frame
    void Update()
    {
        if (Input.GetKey(KeyCode.R) && recording) {
            Debug.Log("Playing...");
            Microphone.End("");
            src.clip = currentAudio;
            src.Play();
            recording = false;      
        } else if (Input.GetKey(KeyCode.L) && !recording){
            Debug.Log("Recording...");
            currentAudio = Microphone.Start("", false, maxClipSize, 44100);
            recording = true;
        }
    }
}
```

- Por otro lado la parte de vídeo consiste en asignar una textura de cámara al material del plano que mencionaba antes. También se obtiene el nombre de la cámara que se está usando por consola. Se dan 3 opciones. La primera es pulsar la S para "encender" la TV (asignar la textura de `WebCamTexture`). En el caso de la P se usa para parar la TV, dejando la TV con una imagen de la última textura recogida desde la cámara. Con la X se toma una captura del fotograma actual y se guarda en una ruta especificada desde el inspector. Para probar, recomiendo poner el directorio actual (`./`) y abrir un explorador de archivos por un lado, para ver como aparecen las imágenes. La idea de las imágenes es crear un objeto de textura del tamaño de un fotograma del vídeo de la cámara. Luego, se le asigna el fotograma actual y posteriormente se escribe como PNG en la ruta especificada.
```cs
public class TV : MonoBehaviour
{
    // Start is called before the first frame update
    public string savePath;
    private Material tvMaterial;
    private WebCamTexture webCamTexture;
    private int captureCounter = 1;
    void Start()
    {
        tvMaterial = GetComponent<Renderer>().material;
        webCamTexture = new WebCamTexture();
        Debug.Log("Camera name: " + webCamTexture.deviceName);
    }

    // Update is called once per frame
    void Update()
    {
        if (Input.GetKey("s")) {
            tvMaterial.mainTexture = webCamTexture;
            webCamTexture.Play();
        }
        if (Input.GetKey("p")) {
            webCamTexture.Stop();
        }
        if (Input.GetKey("x")) {
            Texture2D snapshot = new Texture2D(webCamTexture.width, webCamTexture.height);
            snapshot.SetPixels(webCamTexture.GetPixels());
            snapshot.Apply();
            System.IO.File.WriteAllBytes(savePath + captureCounter.ToString() + ".png", snapshot.EncodeToPNG());
            ++captureCounter;
        }
    }
}
```
