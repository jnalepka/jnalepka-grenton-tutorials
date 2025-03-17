### Integracja z TaHoma Somfy poprzez LocalAPI

W tym tutorialu przedstawiona została możliwość integracji z TaHoma Somfy poprzez LocalAPI zgodnie z https://somfy-developer.github.io/Somfy-TaHoma-Developer-Mode.



##### 1. Pierwsze kroki

1. Skonfiguruj urządzenie TaHoma Switch w aplikacji TaHoma by Somfy.

2. Zaloguj się na koncie Somfy i włącz tryb programisty dla swojego urządzenia TaHoma Switch.

3. Utwórz token dostępu:

   * Zaloguj się na stronie https://dev.duboc.pro/overkiz swoim kontem Somfy.

     ![](1.PNG)

   * Wybierz "I would like to register local API".

   * Na końcu wygenerowane zostaną następujące dane:

     **Service:** local (Local API)
     **Username:** 2017-XXXX-XXXX
     **Password:** 64e5dXXXXXXXXXXXXXXXX

     Naszym tokenem będzie "Password".


##### 2. Wyciągnięcie deviceURL oraz jego dostępnych komend

W tym celu najlepiej użyć aplikacji Postman. 

* `Postman URL`: https://gateway-2001-0001-1891.local:8443/enduser-mobile-web/1/enduserAPI/setup/devices
* `Authorization Bareer`: token (Password)
* `Method`: GET

Należy przeszukać odpowiedź pod kątem `deviceURL` urządzeń, oraz ich dostępnych metod, jak np. `setClosure` do ustawiania wartości procentowej żaluzji, lub `setOrientation` do ustawiania lameli.


##### 3. Wywołanie akcji na urządzeniu przez Gate Http

Obiekt Http_Request należy skonfigurować w następujący sposób:

* `Host`: https://gateway-20XX-XXXX-XXXX.local:8443
* `Path`: /enduser-mobile-web/1/enduserAPI/exec/apply
* `Method`: POST
* `Request/Response Type`: JSON

![](3.PNG)



Skrypt do wywołania przykładowej komendy wygląda następująco:

```lua
local reqHeaders = "Authorization: Bearer <your-token>"
local eventJson = {
		  actions = {{
		      commands = {{
		          name = "setClosure",
		          parameters = { "50" }
		        }},
		      deviceURL = "io://<your-gateway-id>/<device-id>"
		    }}
		}
GATE_HTTP->SomfyTaHoma_Request->SetRequestHeaders(reqHeaders)
GATE_HTTP->SomfyTaHoma_Request->SetRequestBody(eventJson)
GATE_HTTP->SomfyTaHoma_Request->SendRequest()
```



Po prawidłowo wykonanej komendzie TaHoma zwraca status 200 oraz odpowiedź:

```json
"execId": "25f11e6f-41f9-4c27-92ce-65f567ae7112"
```



##### 4. Odpytywanie o stan urządzenia

> UWAGA! 
>
> Znaki specjalne w ścieżce dla deviceURL powinny być zastąpione sekwencjami unikowymi, czyli np. znak `:` -> `%3A`, a znak `/ ` -> `%2F`.  Najlepiej skopiować gotowy string z tego miejsca:
>
> ![](4.PNG)



Obiekt Http_Request należy skonfigurować w następujący sposób:

* `Host`: https://gateway-20XX-XXXX-XXXX.local:8443
* `Path`: /enduser-mobile-web/1/enduserAPI/setup/devices/io%3A%2F%2F{pin}%2F{deviceId}/states`
* `Method`: GET
* `Request/Response Type`: JSON
* `RequestHeaders`: Authorization: Bearer <your-token>



Odpowiedź jaką dostaniemy wygląda mniej więcej tak:

```json
[
  {
    "type": 3,
    "name": "core:NameState",
    "value": "Box"
  },
  {
    "type": 3,
    "name": "core:CountryCodeState",
    "value": "PL"
  },
  {
    "type": 1,
    "name": "internal:LightingLedPodModeState",
    "value": 1
  }
]
```



Zatem przykładowy skrypt analizujący odpowiedź i sprawdzający wartość np. dla `core:ClosureState` wygląda następująco:

```
local reqJson = GATE_HTTP->SomfyTaHoma_GetState_R3->ResponseBody

if GATE_HTTP->SomfyTaHoma_GetState_R3->StatusCode == 200 then
	if reqJson ~= nil then
		for _, entry in ipairs(reqJson) do
		    local name = entry.name
		    local value = entry.value
		    
		    if name == "core:ClosureState" then
		    		local reversedValue = 100 - value
				GATE_HTTP->tahoma_R3_position = reversedValue
			elseif name == "core:SlateOrientationState" then
		    		local calculatedLamel = -0.9 * value + 90
				GATE_HTTP->tahoma_R3_lamel = calculatedLamel
			end
		end
	end
end
```

Skrypt należy podpiąć pod zdarzenie `OnResponse` obiektu `HttpRequest`. Wewnątrz skryptu należy zamieścić logikę według potrzeb.



