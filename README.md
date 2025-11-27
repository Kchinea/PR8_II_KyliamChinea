# PR8_II_KyliamChinea

## Camara

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
