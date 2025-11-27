# PR8_II_KyliamChinea

## Camara

En el apartado de la camara a decir verdad no tuve ninguna complicacion, con el esqueleto que la profesora dejo en el guión y la api de unity fue muy sencillo realizar la práctica.

```csharp
using UnityEngine;
using System.Collections;

public class Camara1 : MonoBehaviour
{
    public WebCamTexture webcamTexture;
    public Renderer renderer;
    void Start()
    {
        string camaraName = WebCamTexture.devices[0].name;
        Debug.Log(camaraName);
        webcamTexture = new WebCamTexture(camaraName);
        renderer = GetComponent<Renderer>();
    }
    void Update()
    {
        // poner lo que se ve en la camara en una textura
        if (Input.GetKey("s")) {
            renderer.material.mainTexture = webcamTexture;
            webcamTexture.Play();
        }
        // parar la grabacion de la camara
        if(Input.GetKey("p")) {
            webcamTexture.Pause();
        }
        //capturar una imagen de la camara y guardarla en un archivo png
        if(Input.GetKey("x")) {
            Texture2D snapshot = new Texture2D(webcamTexture.width, webcamTexture.height);
            snapshot.SetPixels(webcamTexture.GetPixels());
            snapshot.Apply();
            System.IO.File.WriteAllBytes("snapshot.png", snapshot.EncodeToPNG());
        }
    }
}
```
![Camara](./CamaraGIF.gif)


## Microfono

En el caso del micrófono tuve bastantes más problemas, en el primer apartado fue sencillo y muy parecido a como realice el apartado de cámara, no obstante en el apartado de recibir del microfono y conseugir que saliera en tiempo real por el audio fue imposible.
En primer lugar no se si es por mi ordenador, pero el audio se escucha muy saturado. Por otro lado uno de los principales problemas que tuve fue que al intentar aplicar el Microphone.Start() una y otra vez en el Update() no se conseguia escuchar nada, esto creo que sucedio ya que unity tarda un tiempo en poder mandar el audio desde que se enciende el micrófono, al volver al Update() y hacer un Microphone.Start() nuevo nunca daba tiempo, no obstante luego de comprender esto y mucha lucha ya que no fue fácil de darme cuenta, decidí aplicar el Microphone.Start() en el metodo Start(), cosa que acabo en una solución posiblemente valida.
Por otro lado, está opción que conseguí que no sabía si era la esperada, ya que en el enunciado pone que se empieza a recoger en el Start(), pero al darle a la r me salia todo el audio guardado desde que le di a play en unity cosa que no me parecia que estuviera bien.
Despues de estas problematicas, tambien me encontre con que al intentar realizar la practica de otra manera distinta descubrí la posibilidad de aplicarle el loop, esta funcionalidad nos permite que el AudioClip se comporte como un buffer circular, donde el micrófono va sobreescribiendo el audio antiguo a medida que graba el nuevo. Gracias a esto ya no se acumulaba un audio infinito que empezaba en el inicio, sino que solo se conserva el audio de los últimos segundos.
No obstante, esta solución me acarreo más ptroblemas, al leer el clip en tiempo real, a veces el índice del buffer “retrocedía”, devolviendo un audio que ya habia sonado. Esto quería decir que Microphone.GetPosition() podía ser menor que en el frame anterior (por ejemplo, cuando alcanzaba el final del buffer y volvía a cero por el modo loop). Si no comprobaba este caso, el audio se reproducía mal, con saltos y saturación. Para solucionarlo, tuve que añadir una comprobación para detectar cuando el buffer se reiniciaba y evitar leer posiciones de audio inválidas o ya sobrescritas.
Además, en el guión se recomienda una longitud del clip de 10 segundos, pero esto hacía que el retraso fuese mayor y que la reproducción fuese menos controlada. Finalmente ajusté longitudSec = 1, es decir, un buffer de un solo segundo. Esto mejoró muchísimo el comportamiento porque un buffer más pequeño implica:
- Menos retardo entre lo que grabas y lo que escuchas.
- Menos probabilidad de leer datos antiguos.
- Más control al manejar el índice del buffer.
- Mejor rendimiento y menos saturación.
Con está solución de igual forma no pude conseguir el comportamiento que esperaba al incio, ya que se sigue escuchando saturado en algunos casos y pueden llegar a escucharse algún sonido corto que significa que el buffer retrocedió pero me parece presentable y no he podido conseguir una solución mejor.


###   Primer ejercicio
```csharp
using UnityEngine;
public class Sonido : MonoBehaviour
{
    public AudioSource audioSource;

    void Awake()
    {
        audioSource = GetComponent<AudioSource>();
    }

    void OnCollisionEnter(Collision other)
    {
        audioSource.Play();
    }
}

```

### Segundo ejercicio


```csharp
using UnityEngine;

public class Sonido2 : MonoBehaviour
{
    private AudioSource audioSource;
    private string micDevice;
    private bool micOn = false;
    private int lastMicPos = 0;

    void Start()
    {
        audioSource = GetComponent<AudioSource>();
        micDevice = Microphone.devices[0];
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.R) && !micOn)
        {
            audioSource.clip = Microphone.Start(micDevice, true, 1, 44100);
            micOn = true;
            lastMicPos = 0;
        }

        if (micOn && !audioSource.isPlaying)
        {
            if (Microphone.GetPosition(micDevice) > 0)
            {
                audioSource.Play();
                audioSource.timeSamples = Microphone.GetPosition(micDevice);
            }
        }

        if (micOn && audioSource.isPlaying)
        {
            int pos = Microphone.GetPosition(micDevice);
            if (pos == 0 || pos < lastMicPos)
            {
                // Adelantamos la reproducción al punto actual para evitar repetir audio viejo
                audioSource.timeSamples = pos;
            }
            lastMicPos = pos;
        }
        if (Input.GetKeyDown(KeyCode.T) && micOn)
        {
            Microphone.End(micDevice);
            audioSource.Stop();
            audioSource.clip = null;
            micOn = false;
            lastMicPos = 0;
        }
    }
}
```


