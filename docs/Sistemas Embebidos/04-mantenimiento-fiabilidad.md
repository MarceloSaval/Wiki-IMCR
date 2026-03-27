---
title: "4. Mantenimiento y fiabilidad"
---
# 4. El Enfoque IMCR: Mantenimiento en Producción

*(Escribe un párrafo introductorio: una vez que el ESP32 está programado y desplegado (ej. en el techo de una nave industrial), el mantenimiento presencial es costoso. Debemos diseñar pensando en la tolerancia a fallos).*

## Mantenimiento Autónomo: Watchdog Timer (WDT)
*(Explica qué es el Watchdog: un temporizador por hardware que, si el software se cuelga y no lo "alimenta", reinicia el sistema automáticamente. Es clave para el mantenimiento correctivo sin intervención humana).*

## Mantenimiento Evolutivo: Actualizaciones OTA
*(Explica que las actualizaciones Over-The-Air permiten flashear un nuevo firmware vía Wi-Fi, eliminando la necesidad de acceder físicamente al puerto serie del dispositivo).*

## Fiabilidad del Sistema
Desde la ingeniería de mantenimiento, asumiendo una tasa de fallos constante ($\lambda$), la fiabilidad $R(t)$ del dispositivo se calcula como:

$$R(t) = e^{-\lambda t}$$

Implementar OTA y WDT no reduce los fallos físicos de hardware, pero **reduce drásticamente el Tiempo Medio de Reparación (MTTR)** ante fallos de software, aumentando la disponibilidad y fiabilidad global de la red de sistemas embebidos.

[⬅ Anterior: Entornos](./03-entornos-desarrollo.md) | [Volver al Índice](./00-index.md)