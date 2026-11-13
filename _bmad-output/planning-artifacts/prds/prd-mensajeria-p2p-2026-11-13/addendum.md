---
title: "Addendum: PRD mensajeria-p2p"
status: draft
created: 2026-11-13
updated: 2026-11-13
---

# Addendum — profundidad para arquitectura

> Manda `docs/continuidad-proyecto.md`; las referencias «§» apuntan a él. Nada de lo recogido aquí es una decisión salvo que lleve la etiqueta DECISIÓN APROBADA.

## A. Mecanismo de emparejamiento presencial (DECISIÓN PENDIENTE, para arquitectura)

Aportado por Salvador el 2026-11-13 durante el PRD. No elige mecanismo; fija el criterio y nombra candidatos.

- Criterio de Salvador: «algo clásico pero funcional y seguro».
- Restricción reafirmada por Salvador: el emparejamiento es presencial —las dos personas juntas, cada una con su equipo Linux— sea por código, QR u otro mecanismo. Un mecanismo que permita emparejar a distancia queda excluido (§5.2).
- Candidato: QR. Objeción de Salvador: leer un QR con un ordenador es complicado.
- Candidato: código (introducido o comparado entre los dos equipos).
- Candidato: localización por proximidad del otro dispositivo con la aplicación de consola abierta, una vez lanzado el comando.
- HECHO: §5.2 exige protección frente a MITM y replay, confirmación bilateral y vinculación criptográfica de las identidades con la información de conectividad; advierte de que un QR por sí solo no demuestra proximidad física.

## B. Proceso de fondo: formas de ejecución (PROPUESTA, para arquitectura)

El PRD fija la capacidad (DECISIÓN APROBADA por Salvador el 2026-11-13: existe un proceso de fondo propio de la aplicación que el usuario arranca y para sin root). El cómo queda para arquitectura.

- Idea de Salvador: en Linux podría ser un servicio siempre activo que se pueda arrancar y parar.
- HECHO: un servicio del sistema exige root (es lo que hace Enola, §14) y choca con la decisión «sin root» (§14–15). Los servicios de usuario de systemd (`systemctl --user`) se instalan en la carpeta del usuario y se gestionan sin root (`systemd.unit(5)`, `systemctl(1)`).
- HIPÓTESIS sin verificar: mantener el servicio vivo sin sesión abierta requiere «linger» (`loginctl enable-linger`), que según la distribución puede pedir credenciales de administrador.
- HECHO: no todas las distribuciones Linux usan systemd; la ejecución en primer plano en una terminal debe bastar por sí sola.
- Consecuencia de producto aceptada por Salvador: con arranque automático, «en línea» significa «equipo encendido con sesión iniciada», no «usando la aplicación». El emisor no distingue cómo se ejecuta la aplicación del destinatario; solo si es alcanzable.
