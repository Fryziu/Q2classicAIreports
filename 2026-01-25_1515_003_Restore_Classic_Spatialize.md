# Q2classic - Przywrócenie Klasycznej Implementacji S_SpatializeOrigin
**Data:** 2026-01-25  
**Status:** ✅ Zakończone sukcesem

## Zmiany
**Plik:** `src/client/snd_dma.c:851-948`

### Usunięto
- Nową, złożoną implementację z `s_quality->integer`
- Dynamiczne modele tłumienia (poziom 1, 2)
- Efekty dodatkowe (oś Z, kwadratowy spadek)
- Logikę z `attenuation_factor`

### Przywrócono
- Klasyczną implementację Quake II zgodną z R1Q2-tsdmod
- Prosty algorytm attenuation: `dist -= SOUND_FULLVOLUME; dist *= dist_mult;`
- Tradycyjne panning: `rscale = 0.5 * (1.0 + dot); lscale = 0.5 * (1.0 - dot);`
- Stan `cls.state != ca_active` dla pełnej głości w menu

## Wyniki
- **Kompilacja:** ✅ Poprawna
- **Zgodność:** Teraz w pełni zgodne z oryginalnym Quake 2
- **ATTN_NORM/ATTN_NONE/ATTN_STATIC:** Działają zgodnie z klasycznym modelem

## Test
Dźwięki powinny teraz zachowywać się identycznie jak w oryginalnym Quake 2:
- `ATTN_NONE`: Pełna głośność bez tłumienia
- `ATTN_NORM`: Standardowe tłumienie 0.0005
- `ATTN_STATIC`: Szybkie tłumienie 0.001