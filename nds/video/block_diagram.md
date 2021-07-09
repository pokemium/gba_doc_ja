# Display System Block Diagram

```
             _____________               __________
  VRAM A -->| 2D Graphics |--------OBJ->|          |
  VRAM B -->| Engine A    |--------BG3->| Layering |
  VRAM C -->|             |--------BG2->| and      |
  VRAM D -->|             |--------BG1->| Special  |
  VRAM E -->|             |   ___       | Effects  |
  VRAM F -->|             |->|SEL|      |          |          ______
  VRAM G -->| - - - - - - |  |BG0|-BG0->|          |----o--->|      |
            | 3D Graphics |->|___|      |__________|    |    |Select|
            | Engine      |                             |    |Video |
            |_____________|--------3D----------------.  |    |Input |
             _______      _______              ___   |  |    |      |
            |       |    |       |<-----------|SEL|<-'  |    |and   |-->
            |       |    |       |    _____   |A  |     |    |      |
  VRAM A <--|Select |    |Select |   |     |<-|___|<----'    |Master|
  VRAM B <--|Capture|<---|Capture|<--|Blend|   ___           |Bright|
  VRAM C <--|Dest.  |    |Source |   |_____|<-|SEL|<----.    |A     |
  VRAM D <--|       |    |       |            |B  |     |    |      |
            |_______|    |_______|<-----------|___|<-.  |    |      |
             _______                                 |  |    |      |
  VRAM A -->|Select |                                |  |    |      |
  VRAM B -->|Display|--------------------------------o------>|      |
  VRAM C -->|VRAM   |                                   |    |      |
  VRAM D -->|_______|   _____________                   |    |      |
                       |Main Memory  |                  |    |      |
  Main   ------DMA---->|Display FIFO |------------------o--->|______|
  Memory               |_____________|
             _____________               __________           ______
  VRAM C -->| 2D Graphics |--------OBJ->| Layering |         |      |
  VRAM D -->| Engine B    |--------BG3->| and      |         |Master|
  VRAM H -->|             |--------BG2->| Special  |-------->|Bright|-->
  VRAM I -->|             |--------BG1->| Effects  |         |B     |
            |_____________|--------BG0->|__________|         |______|
```
