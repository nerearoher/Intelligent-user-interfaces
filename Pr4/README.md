# Pr谩ctica 4: Micr贸fono y c谩mara 馃搶

### Esta pr谩ctica tiene objetivo aplicar interfaces naturales en componentes con Unity3D (c谩mara y micr贸fono). 
-----------------------------------
<br />

### **Primer ejercicio - Micr贸fono**

Se ha implementado un script, incluido en el [repositorio](./src/MicrophoneController.cs), para la obtenci贸n de sonido desde el micr贸fono del ordenador. Este script fue asignado a un objeto destinado para el control de las reglas del juego, un GameObject al que hemos denominado `GameManager`. Para permitir un uso b谩sico del micr贸fono, se implement贸 la siguiente funcionalidad en el script `MicrophoneController.cs` asociado a este:

```csharp
public class MicrophoneController : MonoBehaviour
{
    private AudioClip audioClip;

    void Start()
    {
        int min, max = 0;
        Microphone.GetDeviceCaps(null, out min, out max);
        Debug.Log(string.Format("Recording... (freq: {0})", max));
        audioClip = Microphone.Start(Microphone.devices[0], true, 999, 44100);
    }
```

Cuando comienza la simulaci贸n, el script comienza a grabar el sonido ambiente utilizando el micr贸fono seleccionado por defecto por el sistema operativo. Para permitir al usuario interactuar con el juego a trav茅s de este medio, decidimos que la mejor forma ser铆a lanzar un evento cuando se detectase que el jugador emitiese un ruido lo suficientemente alto. Este evento luego ser铆a conectado a otra funcionalidad en el juego, permitiendo una interacci贸n m谩s natural con el mismo. Para implementar esta funcionalidad, creamos el siguiente contenido en el m茅todo `Update()`:

```csharp
    public static event MicrophoneDelegate loudEvent;
    public static float MicVolume;
    public float threshold = -50f;

    void Update()
    {
        MicVolume = 20 * Mathf.Log10(Mathf.Abs(calculateVolume()));
        if (MicVolume > threshold) loudEvent();        
    }
```
![imagen_door](./img/img1.PNG)

En cada frame del juego, se calcular谩 el nivel de ruido capturado por el micr贸fono. Si este ruido supera un cierto umbral, el evento es disparado. Para calcular el nivel de ruido (luego convertido a decibelios), se utiliz贸 el m茅todo `calculateVolume()`:

```csharp
    private int sampleWindow = 128;

    float calculateVolume()
    {
        float levelMax = 0;
        float[] waveData = new float[sampleWindow];
        int micPosition = Microphone.GetPosition(null) - (sampleWindow + 1);
        if (micPosition < 0) return 0;
        audioClip.GetData(waveData, micPosition);
        // Getting a peak on the last 128 samples
        for (int i = 0; i < sampleWindow; i++) {
            float wavePeak = waveData[i] * waveData[i];
            if (levelMax < wavePeak) {
                levelMax = wavePeak;
            }
        }
        return levelMax;
    }
```

El evento creado fue posteriormente conectado con la animaci贸n de una puerta colocada en la escena. Esta puerta cuenta con una animaci贸n de apertura y de cerrado, la cual es activada cuando se dispara el evento asociado con la intensidad del volumen:

```csharp
    private Animation animation;
    private BoxCollider box;
    private bool isOpen;

    void Awake()
    {
        animation = GetComponent<Animation>();
        box = GetComponent<BoxCollider>();
    }

    void Start()
    {
        MicrophoneController.loudEvent += openDoor;
    }

    void openDoor()
    {
        if (!isOpen) 
        {
            isOpen = true;
            animation.Play("Door_Open");
            box.enabled = false;
            
        } 
        else if (isOpen)
        {
            isOpen = false;
            animation.Play("Door_Close");
            box.enabled = true;
        }
    }
```
![imagen_door](./img/img2.PNG)
### **Segundo ejercicio - C谩mara**

Para la obtenci贸n de im谩genes de la c谩mara del ordenador, se ha implementado un script que est谩 incluido en el [repositorio](./src/WebcamController.cs). Este script utiliza la clase `WebCamTexture` que nos proporciona la propia plataforma de Unity, que realiza la traducci贸n de la informaci贸n obtenida de la c谩mara del ordenador a texturas 2D. Adem谩s, el script se le ha asignado a un objeto 3D cubo:

![imagen_cara](./img/img3.PNG)

Para el renderizado de la captura de im谩genes se ha a帽adido un atributo privado de la clase `Renderer`

```csharp
public class WebcamController : MonoBehaviour
{

    private Renderer render;

    void Awake() 
    {
        render = GetComponent<Renderer>();
    }

    void Start()
    {
        WebCamTexture webcam = new WebCamTexture();
        render.material.mainTexture = webcam;
        webcam.Play();
    }
}
```

### **Funcionamiento**

El funcionamiento de la funcionalidad desarollada puede verse en el siguiente v铆deo:

[![Miniatura del v铆deo](./img/img4.PNG)](https://youtu.be/xYisqJESGj8)
