# SignalR con .Net

SignalR es una librería de ASP.NET Core que permite comunicación en tiempo real entre el servidor y los clientes (por ejemplo, una página web).

Sirve para:
- Crear chats en tiempo real
- Notificaciones instantáneas (como en Facebook o WhatsApp)
- Monitoreo de datos en vivo (como dashboards)

## Crear proyecto base

Abre la terminal y crea la carpeta del proyecto:

```bash
bash

mkdir SignalRChat
cd SignalRChat

```

Crea el proyecto Razor Pages:

```bash
bash

dotnet new webapp

```

Esto crea la estructura base con:

- `/Pages/Index.cshtml` — página principal (Index)
- `/wwwroot` — carpeta para archivos estáticos
- `Program.cs` — punto de entrada
- `SignalRChat.csproj` — archivo proyecto

---

## Agregar SignalR al proyecto

Desde la terminal (en la raíz del proyecto):

```bash
bash

dotnet add package Microsoft.AspNetCore.SignalR

```

## **Creación de un concentrador de SignalR**

En la carpeta `Hubs`, cree la clase `ChatHub` con el código siguiente:

C#

```csharp
using Microsoft.AspNetCore.SignalR;

namespace SignalRChat.Hubs
{
    public class ChatHub : Hub
    {
        public async Task SendMessage(string user, string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", user, message);
        }
    }
}

```

La clase `ChatHub` hereda de la clase SignalR[Hub](https://learn.microsoft.com/es-es/dotnet/api/microsoft.aspnetcore.signalr.hub). La clase `Hub` administra las conexiones, los grupos y la mensajería.

Puede llamarse al método `SendMessage` mediante un cliente conectado para enviar un mensaje a todos los clientes. El código de cliente de JavaScript que llama al método se muestra más adelante en el tutorial. El código de SignalR es asincrónico para proporcionar la máxima escalabilidad.

**Configuración de SignalR**

El servidor de SignalR debe estar configurado para pasar solicitudes de SignalR a SignalR. Agregue el siguiente código resaltado al archivo `Program.cs`.

C#

```csharp
using SignalRChat.Hubs;var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddRazorPages();
builder.Services.AddSignalR();var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapRazorPages();
app.MapHub<ChatHub>("/chatHub");
app.Run();

```

El código resaltado anterior agrega SignalR a los sistemas de inserción de dependencias y enrutamiento de ASP.NET Core.

**Adición del código del cliente de SignalR**

Reemplace el contenido de `Pages/Index.cshtml` por el código siguiente:

CSHTML

```html
@page
<div class="container">
    <div class="row p-1">
        <div class="col-1">User</div>
        <div class="col-5"><input type="text" id="userInput" /></div>
    </div>
    <div class="row p-1">
        <div class="col-1">Message</div>
        <div class="col-5"><input type="text" class="w-100" id="messageInput" /></div>
    </div>
    <div class="row p-1">
        <div class="col-6 text-end">
            <input type="button" id="sendButton" value="Send Message" />
        </div>
    </div>
    <div class="row p-1">
        <div class="col-6">
            <hr />
        </div>
    </div>
    <div class="row p-1">
        <div class="col-6">
            <ul id="messagesList"></ul>
        </div>
    </div>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/7.0.5/signalr.min.js"></script>
<script src="~/js/chat.js"></script>

```

El marcado anterior:

- Crea cuadros de texto y un botón Enviar.
- Crea una lista con `id="messagesList"` para mostrar los mensajes que se reciben desde el concentrador de SignalR.
- Incluye las referencias de script a SignalR y el código de aplicación de `chat.js` se crea en el paso siguiente.

En la carpeta `wwwroot/js`, cree un archivo `chat.js` con el código siguiente:

JavaScript

```javascript
"use strict";

var connection = new signalR.HubConnectionBuilder().withUrl("/chatHub").build();

//Disable the send button until connection is established.
document.getElementById("sendButton").disabled = true;

connection.on("ReceiveMessage", function (user, message) {
    var li = document.createElement("li");
    document.getElementById("messagesList").appendChild(li);
    // We can assign user-supplied strings to an element's textContent because it
    // is not interpreted as markup. If you're assigning in any other way, you
    // should be aware of possible script injection concerns.
    li.textContent = `${user} says ${message}`;
});

connection.start().then(function () {
    document.getElementById("sendButton").disabled = false;
}).catch(function (err) {
    return console.error(err.toString());
});

document.getElementById("sendButton").addEventListener("click", function (event) {
    var user = document.getElementById("userInput").value;
    var message = document.getElementById("messageInput").value;
    connection.invoke("SendMessage", user, message).catch(function (err) {
        return console.error(err.toString());
    });
    event.preventDefault();
});

```

JavaScript anterior:

- Crea e inicia una conexión.
- Agrega al botón de envío un controlador que envía mensajes al concentrador.
- Agrega al objeto de conexión un controlador que recibe mensajes desde el concentrador y los agrega a la lista.