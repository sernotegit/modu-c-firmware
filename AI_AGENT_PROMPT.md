# MODU-C ZMK Firmware AI Setup Prompt

이 문서는 `modu-c-firmware` 저장소를 포크한 후, AI 에이전트(Antigravity, Cursor, Claude Code, ChatGPT 등)에 전달하여 **GitHub Actions 펌웨어 자동 빌드** 및 **ZMK Keymap Editor 웹 편집기 지원** 설정을 자동으로 완성할 수 있도록 작성된 프롬프트입니다.

---

## 📋 AI 에이전트에 전달할 프롬프트

```markdown
이 저장소는 MODU-C 키보드용 ZMK 펌웨어 소스(https://github.com/22sh22/modu-c-firmware)를 포크한 저장소입니다.
현재 저장소에는 GitHub Actions 자동 빌드 워크플로우와 ZMK Keymap Editor(https://nickcoutsos.github.io/keymap-editor/) 지원에 필요한 표준 ZMK config 구조가 없습니다.

저장소 루트에 아래 파일들을 생성하고 GitHub에 커밋 및 푸시해 주세요.
(※ 모든 레이아웃 메타데이터에는 Keymap Editor가 요구하는 `row`, `col`, `matrix`, `x`, `y` 속성이 모두 포함되어야 합니다.)

---

### 1. `build.yaml` (저장소 최상위 루트)
```yaml
include:
  - board: ms88sf3/nrf52840
    shield: modu_left
  - board: ms88sf3/nrf52840
    shield: modu_right
```

---

### 2. `.github/workflows/build.yml`
```yaml
name: Build MODU Firmware

on:
  push:
    paths:
      - 'config/**'
      - 'modu-module/**'
      - 'zmk-pmw3610-driver/**'
      - '.github/workflows/**'
      - 'build.yaml'
  pull_request:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: zmkfirmware/zmk-build-arm:stable
    strategy:
      fail-fast: false
      matrix:
        include:
          - shield: modu_left
          - shield: modu_right
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Cache west modules
        uses: actions/cache@v4
        env:
          cache-name: cache-zephyr-modules
        with:
          path: |
            modules/
            tools/
            zephyr/
            zmk/
            bootloader/
          key: ${{ runner.os }}-build-${{ env.cache-name }}-${{ hashFiles('config/west.yml', 'modu-module/**', 'zmk-pmw3610-driver/**') }}
          restore-keys: |
            ${{ runner.os }}-build-${{ env.cache-name }}-

      - name: West Init
        run: |
          if [ ! -d ".west" ]; then
            west init -l config
          fi

      - name: West Update
        run: |
          west update
          west zephyr-export

      - name: West Build (${{ matrix.shield }})
        run: |
          west build -s zmk/app -d build/${{ matrix.shield }} -b ms88sf3/nrf52840 -- \
            -DZMK_CONFIG="${GITHUB_WORKSPACE}/config" \
            -DZMK_EXTRA_MODULES="${GITHUB_WORKSPACE}/modu-module;${GITHUB_WORKSPACE}/zmk-pmw3610-driver" \
            -DSHIELD=${{ matrix.shield }}

      - name: Convert to UF2
        run: |
          python3 tools/uf2/uf2conv.py -f 0xADA52840 -c -o ${{ matrix.shield }}.uf2 build/${{ matrix.shield }}/zephyr/zmk.hex

      - name: Upload UF2 Artifact
        uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.shield }}
          path: ${{ matrix.shield }}.uf2
```

---

### 3. `config/west.yml`
```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
  self:
    path: config
```

---

### 4. `config/info.json` 및 `config/modu.json`
(두 파일 모두 동일한 내용으로 생성해 주세요.)
```json
{
  "keyboard_name": "Modu",
  "url": "https://github.com/22sh22/modu-c-firmware",
  "maintainer": "Ryu",
  "width": 16,
  "height": 6.5,
  "layouts": {
    "LAYOUT": {
      "layout": [
        {"row": 0, "col": 0, "x": 0, "y": 0, "matrix": [0, 0]},
        {"row": 0, "col": 1, "x": 1, "y": 0, "matrix": [0, 1]},
        {"row": 0, "col": 2, "x": 2, "y": 0, "matrix": [0, 2]},
        {"row": 0, "col": 3, "x": 3, "y": 0, "matrix": [0, 3]},
        {"row": 0, "col": 4, "x": 4, "y": 0, "matrix": [0, 4]},
        {"row": 0, "col": 5, "x": 5, "y": 0, "matrix": [0, 5]},
        {"row": 0, "col": 6, "x": 8, "y": 0, "matrix": [0, 6]},
        {"row": 0, "col": 7, "x": 9, "y": 0, "matrix": [0, 7]},
        {"row": 0, "col": 8, "x": 10, "y": 0, "matrix": [0, 8]},
        {"row": 0, "col": 9, "x": 11, "y": 0, "matrix": [0, 9]},
        {"row": 0, "col": 10, "x": 12, "y": 0, "matrix": [0, 10]},
        {"row": 0, "col": 11, "x": 13, "y": 0, "matrix": [0, 11]},

        {"row": 1, "col": 0, "x": 0, "y": 1, "matrix": [1, 0]},
        {"row": 1, "col": 1, "x": 1, "y": 1, "matrix": [1, 1]},
        {"row": 1, "col": 2, "x": 2, "y": 1, "matrix": [1, 2]},
        {"row": 1, "col": 3, "x": 3, "y": 1, "matrix": [1, 3]},
        {"row": 1, "col": 4, "x": 4, "y": 1, "matrix": [1, 4]},
        {"row": 1, "col": 5, "x": 5, "y": 1, "matrix": [1, 5]},
        {"row": 1, "col": 6, "x": 8, "y": 1, "matrix": [1, 6]},
        {"row": 1, "col": 7, "x": 9, "y": 1, "matrix": [1, 7]},
        {"row": 1, "col": 8, "x": 10, "y": 1, "matrix": [1, 8]},
        {"row": 1, "col": 9, "x": 11, "y": 1, "matrix": [1, 9]},
        {"row": 1, "col": 10, "x": 12, "y": 1, "matrix": [1, 10]},
        {"row": 1, "col": 11, "x": 13, "y": 1, "matrix": [1, 11]},

        {"row": 2, "col": 0, "x": 0, "y": 2, "matrix": [2, 0]},
        {"row": 2, "col": 1, "x": 1, "y": 2, "matrix": [2, 1]},
        {"row": 2, "col": 2, "x": 2, "y": 2, "matrix": [2, 2]},
        {"row": 2, "col": 3, "x": 3, "y": 2, "matrix": [2, 3]},
        {"row": 2, "col": 4, "x": 4, "y": 2, "matrix": [2, 4]},
        {"row": 2, "col": 5, "x": 5, "y": 2, "matrix": [2, 5]},
        {"row": 2, "col": 6, "x": 8, "y": 2, "matrix": [2, 6]},
        {"row": 2, "col": 7, "x": 9, "y": 2, "matrix": [2, 7]},
        {"row": 2, "col": 8, "x": 10, "y": 2, "matrix": [2, 8]},
        {"row": 2, "col": 9, "x": 11, "y": 2, "matrix": [2, 9]},
        {"row": 2, "col": 10, "x": 12, "y": 2, "matrix": [2, 10]},
        {"row": 2, "col": 11, "x": 13, "y": 2, "matrix": [2, 11]},

        {"row": 3, "col": 0, "x": 0, "y": 3, "matrix": [3, 0]},
        {"row": 3, "col": 1, "x": 1, "y": 3, "matrix": [3, 1]},
        {"row": 3, "col": 2, "x": 2, "y": 3, "matrix": [3, 2]},
        {"row": 3, "col": 3, "x": 3, "y": 3, "matrix": [3, 3]},
        {"row": 3, "col": 4, "x": 4, "y": 3, "matrix": [3, 4]},
        {"row": 3, "col": 5, "x": 5, "y": 3, "matrix": [3, 5]},
        {"row": 3, "col": 6, "x": 8, "y": 3, "matrix": [3, 6]},
        {"row": 3, "col": 7, "x": 9, "y": 3, "matrix": [3, 7]},
        {"row": 3, "col": 8, "x": 10, "y": 3, "matrix": [3, 8]},
        {"row": 3, "col": 9, "x": 11, "y": 3, "matrix": [3, 9]},
        {"row": 3, "col": 10, "x": 12, "y": 3, "matrix": [3, 10]},
        {"row": 3, "col": 11, "x": 13, "y": 3, "matrix": [3, 11]},

        {"row": 4, "col": 0, "x": 0, "y": 4, "matrix": [4, 0]},
        {"row": 4, "col": 1, "x": 1, "y": 4, "matrix": [4, 1]},
        {"row": 4, "col": 2, "x": 2, "y": 4, "matrix": [4, 2]},
        {"row": 4, "col": 3, "x": 3, "y": 4, "matrix": [4, 3]},
        {"row": 4, "col": 4, "x": 4, "y": 4, "matrix": [4, 4]},
        {"row": 4, "col": 5, "x": 5, "y": 4, "matrix": [4, 5]},
        {"row": 4, "col": 6, "x": 8, "y": 4, "matrix": [4, 6]},
        {"row": 4, "col": 7, "x": 9, "y": 4, "matrix": [4, 7]},
        {"row": 4, "col": 8, "x": 10, "y": 4, "matrix": [4, 8]},
        {"row": 4, "col": 9, "x": 11, "y": 4, "matrix": [4, 9]},
        {"row": 4, "col": 10, "x": 12, "y": 4, "matrix": [4, 10]},
        {"row": 4, "col": 11, "x": 13, "y": 4, "matrix": [4, 11]},

        {"row": 5, "col": 0, "x": 3.5, "y": 5.2, "matrix": [5, 0]},
        {"row": 5, "col": 1, "x": 4.5, "y": 5.2, "matrix": [5, 1]},
        {"row": 5, "col": 2, "x": 5.5, "y": 5.2, "matrix": [5, 2]},

        {"row": 5, "col": 6, "x": 8.0, "y": 5.2, "matrix": [5, 6]},
        {"row": 5, "col": 7, "x": 9.0, "y": 5.2, "matrix": [5, 7]},
        {"row": 5, "col": 8, "x": 10.0, "y": 5.2, "matrix": [5, 8]},
        {"row": 5, "col": 9, "x": 11.0, "y": 5.2, "matrix": [5, 9]}
      ]
    },
    "default_transform": {
      "layout": [
        {"row": 0, "col": 0, "x": 0, "y": 0, "matrix": [0, 0]},
        {"row": 0, "col": 1, "x": 1, "y": 0, "matrix": [0, 1]},
        {"row": 0, "col": 2, "x": 2, "y": 0, "matrix": [0, 2]},
        {"row": 0, "col": 3, "x": 3, "y": 0, "matrix": [0, 3]},
        {"row": 0, "col": 4, "x": 4, "y": 0, "matrix": [0, 4]},
        {"row": 0, "col": 5, "x": 5, "y": 0, "matrix": [0, 5]},
        {"row": 0, "col": 6, "x": 8, "y": 0, "matrix": [0, 6]},
        {"row": 0, "col": 7, "x": 9, "y": 0, "matrix": [0, 7]},
        {"row": 0, "col": 8, "x": 10, "y": 0, "matrix": [0, 8]},
        {"row": 0, "col": 9, "x": 11, "y": 0, "matrix": [0, 9]},
        {"row": 0, "col": 10, "x": 12, "y": 0, "matrix": [0, 10]},
        {"row": 0, "col": 11, "x": 13, "y": 0, "matrix": [0, 11]},

        {"row": 1, "col": 0, "x": 0, "y": 1, "matrix": [1, 0]},
        {"row": 1, "col": 1, "x": 1, "y": 1, "matrix": [1, 1]},
        {"row": 1, "col": 2, "x": 2, "y": 1, "matrix": [1, 2]},
        {"row": 1, "col": 3, "x": 3, "y": 1, "matrix": [1, 3]},
        {"row": 1, "col": 4, "x": 4, "y": 1, "matrix": [1, 4]},
        {"row": 1, "col": 5, "x": 5, "y": 1, "matrix": [1, 5]},
        {"row": 1, "col": 6, "x": 8, "y": 1, "matrix": [1, 6]},
        {"row": 1, "col": 7, "x": 9, "y": 1, "matrix": [1, 7]},
        {"row": 1, "col": 8, "x": 10, "y": 1, "matrix": [1, 8]},
        {"row": 1, "col": 9, "x": 11, "y": 1, "matrix": [1, 9]},
        {"row": 1, "col": 10, "x": 12, "y": 1, "matrix": [1, 10]},
        {"row": 1, "col": 11, "x": 13, "y": 1, "matrix": [1, 11]},

        {"row": 2, "col": 0, "x": 0, "y": 2, "matrix": [2, 0]},
        {"row": 2, "col": 1, "x": 1, "y": 2, "matrix": [2, 1]},
        {"row": 2, "col": 2, "x": 2, "y": 2, "matrix": [2, 2]},
        {"row": 2, "col": 3, "x": 3, "y": 2, "matrix": [2, 3]},
        {"row": 2, "col": 4, "x": 4, "y": 2, "matrix": [2, 4]},
        {"row": 2, "col": 5, "x": 5, "y": 2, "matrix": [2, 5]},
        {"row": 2, "col": 6, "x": 8, "y": 2, "matrix": [2, 6]},
        {"row": 2, "col": 7, "x": 9, "y": 2, "matrix": [2, 7]},
        {"row": 2, "col": 8, "x": 10, "y": 2, "matrix": [2, 8]},
        {"row": 2, "col": 9, "x": 11, "y": 2, "matrix": [2, 9]},
        {"row": 2, "col": 10, "x": 12, "y": 2, "matrix": [2, 10]},
        {"row": 2, "col": 11, "x": 13, "y": 2, "matrix": [2, 11]},

        {"row": 3, "col": 0, "x": 0, "y": 3, "matrix": [3, 0]},
        {"row": 3, "col": 1, "x": 1, "y": 3, "matrix": [3, 1]},
        {"row": 3, "col": 2, "x": 2, "y": 3, "matrix": [3, 2]},
        {"row": 3, "col": 3, "x": 3, "y": 3, "matrix": [3, 3]},
        {"row": 3, "col": 4, "x": 4, "y": 3, "matrix": [3, 4]},
        {"row": 3, "col": 5, "x": 5, "y": 3, "matrix": [3, 5]},
        {"row": 3, "col": 6, "x": 8, "y": 3, "matrix": [3, 6]},
        {"row": 3, "col": 7, "x": 9, "y": 3, "matrix": [3, 7]},
        {"row": 3, "col": 8, "x": 10, "y": 3, "matrix": [3, 8]},
        {"row": 3, "col": 9, "x": 11, "y": 3, "matrix": [3, 9]},
        {"row": 3, "col": 10, "x": 12, "y": 3, "matrix": [3, 10]},
        {"row": 3, "col": 11, "x": 13, "y": 3, "matrix": [3, 11]},

        {"row": 4, "col": 0, "x": 0, "y": 4, "matrix": [4, 0]},
        {"row": 4, "col": 1, "x": 1, "y": 4, "matrix": [4, 1]},
        {"row": 4, "col": 2, "x": 2, "y": 4, "matrix": [4, 2]},
        {"row": 4, "col": 3, "x": 3, "y": 4, "matrix": [4, 3]},
        {"row": 4, "col": 4, "x": 4, "y": 4, "matrix": [4, 4]},
        {"row": 4, "col": 5, "x": 5, "y": 4, "matrix": [4, 5]},
        {"row": 4, "col": 6, "x": 8, "y": 4, "matrix": [4, 6]},
        {"row": 4, "col": 7, "x": 9, "y": 4, "matrix": [4, 7]},
        {"row": 4, "col": 8, "x": 10, "y": 4, "matrix": [4, 8]},
        {"row": 4, "col": 9, "x": 11, "y": 4, "matrix": [4, 9]},
        {"row": 4, "col": 10, "x": 12, "y": 4, "matrix": [4, 10]},
        {"row": 4, "col": 11, "x": 13, "y": 4, "matrix": [4, 11]},

        {"row": 5, "col": 0, "x": 3.5, "y": 5.2, "matrix": [5, 0]},
        {"row": 5, "col": 1, "x": 4.5, "y": 5.2, "matrix": [5, 1]},
        {"row": 5, "col": 2, "x": 5.5, "y": 5.2, "matrix": [5, 2]},

        {"row": 5, "col": 6, "x": 8.0, "y": 5.2, "matrix": [5, 6]},
        {"row": 5, "col": 7, "x": 9.0, "y": 5.2, "matrix": [5, 7]},
        {"row": 5, "col": 8, "x": 10.0, "y": 5.2, "matrix": [5, 8]},
        {"row": 5, "col": 9, "x": 11.0, "y": 5.2, "matrix": [5, 9]}
      ]
    }
  }
}
```

---

### 5. `config/modu.keymap`
```dts
/*
 * Copyright (c) 2026 EKS Inc.
 * Created by Ryu.
 * SPDX-License-Identifier: LicenseRef-EKS-NonCommercial-1.0
 */

#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include <dt-bindings/zmk/bt.h>
#include <dt-bindings/zmk/pointing.h>

/ {
    keymap {
        compatible = "zmk,keymap";

        default_layer {
// -----------------------------------------------------------------
// |ESC | 1  | 2  | 3  | 4  | 5  |   | 6  | 7  | 8  | 9  | 0  |DEL |
// |TAB | Q  | W  | E  | R  | T  |   | Y  | U  | I  | O  | P  |HOME|
// |CAPS| A  | S  | D  | F  | G  |   | H  | J  | K  | L  | ;  |PGUP|
// |LSFT| Z  | X  | C  | V  | B  |   | N  | M  | ,  | .  | /  |PGDN|
// |LCTL|LALT|LGUI|                                 |RALT|RCTL|END |
//                |MO1 |LANG|SPC |   |BSPC|RET |MO1 | B  |
            bindings = <
&kp ESC   &kp N1   &kp N2    &kp N3    &kp N4    &kp N5      &kp N6    &kp N7    &kp N8    &kp N9    &kp N0    &kp DEL
&kp TAB   &kp Q    &kp W     &kp E     &kp R     &kp T       &kp Y     &kp U     &kp I     &kp O     &kp P     &kp HOME
&kp CAPS  &kp A    &kp S     &kp D     &kp F     &kp G       &kp H     &kp J     &kp K     &kp L     &kp SEMI  &kp PG_UP
&kp LSHFT &kp Z    &kp X     &kp C     &kp V     &kp B       &kp N     &kp M     &kp COMMA &kp DOT   &kp FSLH  &kp PG_DN
&kp LCTRL &kp LALT &kp LGUI  &none     &none     &none       &none     &none     &none     &kp RALT  &kp RCTRL &kp END
                             &kp LANG1 &kp SPACE &mo 1       &kp BSPC  &kp RET   &mo 1     &kp B
            >;
        };

        lower_layer {
            bindings = <
&kp F1     &kp F2       &kp F3       &kp F4       &kp F5     &kp F6       &kp F7        &kp F8       &kp F9       &kp F10      &kp F11             &kp F12
&trans     &kp HOME     &kp UP       &kp END      &trans     &trans       &mkp MCLK     &kp KP_N7    &kp KP_N8    &kp KP_N9    &kp KP_PLUS         &kp BSPC
&trans     &kp LEFT     &kp DOWN     &kp RIGHT    &trans     &trans       &mkp RCLK     &kp KP_N4    &kp KP_N5    &kp KP_N6    &kp KP_MINUS        &kp KP_COMMA
&bt BT_CLR &bt BT_SEL 0 &bt BT_SEL 1 &bt BT_SEL 2 &trans     &trans       &mkp LCLK     &kp KP_N1    &kp KP_N2    &kp KP_N3    &kp KP_MULTIPLY     &kp KP_DOT
&trans     &trans       &trans       &trans       &trans     &trans       &trans        &trans       &trans       &kp KP_N0    &kp KP_DIVIDE       &kp KP_ENTER
                                     &trans       &trans     &trans       &trans        &trans       &trans       &trans
            >;
        };
    };
};
```

---

### 6. `config/modu_left.conf`
```ini
# Copyright (c) 2026 EKS Inc.
# Created by Ryu.
# SPDX-License-Identifier: LicenseRef-EKS-NonCommercial-1.0

CONFIG_ZMK_BLE=y
CONFIG_ZMK_SPLIT=y
CONFIG_ZMK_USB=y

CONFIG_ZMK_KSCAN_MATRIX_POLLING=y
CONFIG_ZMK_KSCAN_DIRECT_POLLING=y

# Trackball (PMW3610)
CONFIG_SPI=y
CONFIG_INPUT=y
CONFIG_ZMK_POINTING=y
CONFIG_PMW3610_ALT=y
CONFIG_PMW3610_ALT_POLL_INTERVAL_MS=15

# Boot-time inner/outer orientation selector on P0.08
CONFIG_MODU_ALT_THUMB_KSCAN=y

# Battery
CONFIG_ZMK_BATTERY_REPORTING=y

# PWM LED breath effect
CONFIG_PWM=y
CONFIG_MODU_LED_BREATH=y
```

---

### 7. `config/modu_right.conf`
```ini
# Copyright (c) 2026 EKS Inc.
# Created by Ryu.
# SPDX-License-Identifier: LicenseRef-EKS-NonCommercial-1.0

CONFIG_ZMK_BLE=y

# Trackball (PMW3610)
CONFIG_SPI=y
CONFIG_INPUT=y
CONFIG_ZMK_POINTING=y
CONFIG_PMW3610_ALT=y
CONFIG_PMW3610_ALT_POLL_INTERVAL_MS=15

# Boot-time inner/outer orientation selector on P0.08
CONFIG_MODU_ALT_THUMB_KSCAN=y

# Battery
CONFIG_ZMK_BATTERY_REPORTING=y

# PWM LED breath effect
CONFIG_PWM=y
CONFIG_MODU_LED_BREATH=y
```

---

위 파일들을 생성한 후, `git add .`, 커밋 및 원격 `main` 브랜치로 `git push`를 진행해 주세요.
```
