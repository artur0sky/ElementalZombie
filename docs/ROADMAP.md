# Roadmap - Elemental Zombie Mod

Este documento detalla las fases de desarrollo para la implementación del mod en **Minecraft Java 1.21.1**, desde la base técnica hasta la pulición final.

## 📍 Fase 1: Cimientos y Registro (Foundation)

**Objetivo**: Tener una entidad funcional en Minecraft aunque no tenga poderes aún.

* [ ] Configuración del entorno Gradle (Forge/Fabric).
* [ ] Registro de la entidad `ElementalZombieEntity`.
* [ ] Creación de assets básicos (Textura base, Modelado).
* [ ] Implementación de IA básica de Zombie (agresión, movimiento).

## 📍 Fase 2: Núcleo de Arquitectura (Core Architecture)

**Objetivo**: Implementar el sistema de capas y el intercambio de estrategias.

* [ ] Crear interfaces de dominio (`ElementalPower`, `BiomeType`).
* [ ] Implementar `ElementalPowerService` y `BiomePowerResolver`.
* [ ] Crear el `MinecraftBiomeAdapter` para conectar con el mundo de MC.
* [ ] Sistema de log para debug de cambios de bioma.

## 📍 Fase 3: Poderes Elementales - Parte 1 (Water & Ice)

**Objetivo**: Implementar los primeros comportamientos dinámicos.

* [ ] **Water Power**:
  * [ ] Pasiva: Respiración y natación fluida.
  * [ ] Básica: Mecánica de drenaje de oxígeno al golpear.
  * [ ] Definitiva: Dash torpedo con knockback masivo.
  * [ ] Debilidad: Sistema de daño por esponjas/cubetas.
* [ ] **Snow Power**:
  * [ ] Pasiva: Contador de ataques para congelación.
  * [ ] Básica: Proyectiles de nieve con daño.
  * [ ] Definitiva: Creación y lanzamiento de bola de nieve gigante.
  * [ ] Debilidad: Vulnerabilidad al fuego y fuentes de calor.

## 📍 Fase 4: Comportamiento Dinámico y Efectos

**Objetivo**: Refinar la transición entre estados y la visualización.

* [ ] Detección de cambio de bioma en tiempo real (Tick-based check).
* [ ] **Mecánica de División (Split)**:
  * [ ] Lógica de detección de salud baja (<20%).
  * [ ] Spawning de mini-zombies elementales.
* [ ] Efectos visuales (Partículas) al cambiar de poder.
* [ ] Cambio de texturas/capas de renderizado según el elemento activo.
* [ ] Sonidos personalizados para cada elemento (golpe torpedo, congelación).

## 📍 Fase 5: Configuración y Balanceo

**Objetivo**: Hacer el mod personalizable y justo para el gameplay.

* [ ] Archivo de configuración `config.json` para ajustar daño y probabilidades.
* [ ] Soporte para biomas de otros mods a través del Resolver.
* [ ] Balanceo de drop de items elementales.
* [ ] Localización (ES/EN).

## 📍 Fase 6: Lanzamiento y Extensión

* [ ] Guía para Colaboradores (Cómo añadir nuevos elementos).
* [ ] Generación de builds de producción.
* [ ] (Opcional) Poderes de Lava, Desierto y Rayo.
