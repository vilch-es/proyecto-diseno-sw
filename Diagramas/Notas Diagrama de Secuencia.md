# Notas de integración

Apuntes que acompañan al análisis del diagrama de secuencia.

## Sincrónico vs asíncrono
 
| Llamada | Tipo | Motivo |
|---|---|---|
| `GET /feed/home` | síncrona | El usuario espera el resultado para continuar |
| `GET /items/{id}/visibility` | síncrona | Validación inmediata requerida antes de reproducir |
| `POST /ad-requests` | síncrona | La decisión de anuncio ocurre antes de iniciar reproducción |
| `POST /playback-sessions` | síncrona | El cliente necesita la URL de reproducción al instante |
| `engagement.signal` | asíncrona | Descubrimiento actualiza el ranking en segundo plano |
| `playback.session.completed` | asíncrona | Señal de watch time para ranking; no bloquea al usuario |
| `ad.revenue.generated` | asíncrona | Monetización atribuye el ingreso eventualmente |
 
## IDs compartidos entre contextos
 
| ID | Dueño | Quién lo usa |
|---|---|---|
| `videoAssetId` | Publicación | Catálogo (vincula asset a ítem), Publicidad (opcional, auditoría), Monetización (opcional) |
| `catalogItemId` | Catálogo | Descubrimiento, Audiencia, Publicidad, Monetización |
| `channelId` | Catálogo | Audiencia, Monetización, Publicidad |
| `userId` | Plataforma compartida (auth) | Todos los contextos |
 
## Consistencia eventual
 
Si Catálogo despublica un video mientras el espectador lo está reproduciendo:
- **Publicación** no interrumpe la sesión activa (la URL de reproducción sigue válida).
- **Catálogo** emite `content.unpublished`.
- **Descubrimiento** recibe la señal vía `POST /signals` y remueve el ítem del índice.
- **Audiencia** recibe el evento y oculta interacciones públicas del contenido.
- **Publicidad** marca el contenido como no elegible para nuevas oportunidades.
 
