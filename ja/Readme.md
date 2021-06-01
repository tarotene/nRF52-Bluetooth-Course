## nRF52 Bluetoothコース

このリポジトリには、nRF52 Bluetooth コースの実践演習が含まれています。コースを終了すると、Nordic nRF5 SDK v15.0.0 にある ble_app_template プロジェクトにカスタム特性を使用して独自のカスタム サービスを作成できるようになります。

このチュートリアルの目的は、ステップ間にあまり理論を入れずに、1 つの特性を持つ 1 つのサービスを作成することです。 ble_app_template プロジェクトでゼロから始めるため、ダウンロードする必要のある .c または .h ファイルはありません。

ただし、コースの手順に従わずに単に例をコンパイルしたい場合は、このリポジトリを SDK v15.0.0/examples/ble_peripheral に複製できます。

<!---
## TODO

- [ ] Add register definition file (.svd) and retarget of printf to the ble_app_uart Segger Embedded Project.
--->

## コース評価

コースの最後にある以下のリンクにあるコース評価フォームに 2 分間で記入してください。

[コース評価フォーム](https://drive.google.com/open?id=1hzn3SHDQ2Qg0d-_jRboxD2qpFKHSPzwKGH7Nz3AKotQ)

コースの教材とプレゼンテーションを改善できるように、コースについて気に入った点/気に入らなかった点をお知らせください。

もちろん評価は匿名です。

## プレゼンテーション

コースのプレゼンテーションは、以下のリンクを使用して PDF 形式でダウンロードできます。

[北欧の紹介](https://github.com/bjornspockeli/custom_ble_service_example/blob/master/pdf/01_Nordic_Company_Introduction.pdf)

[nRF52832 イントロ + 組み込み C イントロ](https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/pdf/02_nRF52%20Student_%20Intro_v0.1.pdf)

[Bluetooth Low Energy プロトコル](https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/pdf/03_Bluetooth%20Overview.pdf)

[ソフトデバイス紹介](https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/pdf/04_Softdevice_Student_Introduction_v0.1.pdf)

## ハードウェア要件

- nRF52 開発キット

## ソフトウェア要件

- nRF5 SDK v15.0.0[ダウンロードページ](http://developer.nordicsemi.com/nRF5_SDK/nRF5_SDK_v15.x.x/)
- 最新バージョンの Segger Embedded Studio[ダウンロード ページ](https://www.segger.com/downloads/embedded-studio/)または最新バージョンの Keil ARM MKD[ダウンロード ページ](https://www.keil.com/demo/eval/arm.htm)または GCC ARM Embedded 6.3 2017-q2-update ツールチェーン[ダウンロード ページ](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads/6-2017-q2-update)
- nRF Connect for Mobile、 [ダウンロードページ](https://www.nordicsemi.com/eng/Products/Nordic-mobile-Apps/nRF-Connect-for-mobile-previously-called-nRF-Master-Control-Panel)
- nRF コマンド ライン ツールの[ダウンロード ページ](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.tools/dita/tools/nrf5x_command_line_tools/nrf5x_installation.html?cp=5_1_1)

## IDE/ツールチェーンのサポート

Nordic Semiconductor は、SDK v14.1.0 で Segger Embedded Studio のサポートを追加し、チュートリアルはその IDE を念頭に置いて書かれています。つまり、メモリ設定/ビルド パラメータを変更する手順は主に SES 向けです。ただし、コードはリスト内の他の IDE/ツールチェーンでコンパイルする必要があります。

- Segger 組み込みスタジオ
- Make と GCC
- キール

## チュートリアルの手順

<!---
```c   
    uint8_t data
    // put C code here
```

[link to webpage](www.google.com)
--->

### ステップ 1 - はじめに

1. ダウンロード ページから nRF5_SDK_15.0.0_17b948a を[ダウンロードし](http://developer.nordicsemi.com/nRF5_SDK/nRF5_SDK_v15.x.x/)、zip をドライブに解凍します (例: C:\NordicSemi\nRF5_SDK_15.0.0_a53641a)。

2. nRF5_SDK_15.0.0_a53641a/examples/ble_peripheral フォルダーに移動し、ble_app_template プロジェクト フォルダーを見つけます。

3. フォルダーのコピーを作成し、 `custom_ble_service_example`という名前を付けます。

4. custom_ble_service_example\pca10040\s132\ses に移動し、ble_app_template_pca10040_s132.emProject プロジェクトを開きます。

### ステップ 2 - カスタム ベース UUID の作成

最初に行う必要があるのは、新しい .c ファイルを作成することです。これを ble_cus.c ( **Cu** stom **S** ervice) と呼び、それに付随する .h ファイル ble_cus.h と呼びます。 main.c ファイルと同じフォルダーに 2 つのファイルを作成します。ヘッダー ファイル ble_cus.h の先頭に、次の .h ファイルを含める必要があります。

```c
/* This code belongs in ble_cus.h*/
#include <stdint.h>
#include <stdbool.h>
#include "ble.h"
#include "ble_srv_common.h"
```

次に、標準化されたプロファイル用に予約されている 16 ビットの Bluetooth SIG UUID のいずれかを使用してサービスを実装しないため、カスタム サービスには 128 ビットの UUID が必要になります。 128 ビットの UUID を生成するにはいくつかの方法がありますが、ここでは[この](https://www.uuidgenerator.net/version4)オンライン UUID ジェネレーターを使用します。 Web ページはランダムな 128 ビット UUID を生成します。私の場合は、

```
f364adc9-b000-4042-ba50-05ca45bf8abc
```

UUID は、UUID の 16 オクテットが 32 桁の 16 進数 (16 進数) として表され、ハイフンで区切られた 5 つのグループに 8-4-4-4-12 の形式で表示されます。 16 オクテットはビッグ エンディアンで指定されますが、SDK ではスモール エンディアン表現を使用します。したがって、次に示すように、ble_cus.h で UUID ベースを定義するときは、バイトの順序を逆にする必要があります。

```c
/* This code belongs in ble_cus.h*/
#define CUSTOM_SERVICE_UUID_BASE         {0xBC, 0x8A, 0xBF, 0x45, 0xCA, 0x05, 0x50, 0xBA, \
                                          0x40, 0x42, 0xB0, 0x00, 0xC9, 0xAD, 0x64, 0xF3}
```

ベース UUID を定義したので、カスタム サービス用の 16 ビット UUID とカスタム値特性用の 16 ビット UUID を定義する必要があります。

```c
/* This code belongs in ble_cus.h*/
#define CUSTOM_SERVICE_UUID               0x1400
#define CUSTOM_VALUE_CHAR_UUID            0x1401
```

ベース UUID に挿入される 16 ビット UUID の値は、ランダムに選択できます。

### ステップ 2 - カスタム サービスの実装

まず最初に、先ほど作成した ble_cus.h ヘッダー ファイルと、ble_cus.c にいくつかの一般的な SDK ヘッダー ファイルをインクルードする必要があります。

```c
/* This code belongs in ble_cus.c*/
#include "sdk_common.h"
#include "ble_srv_common.h"
#include "ble_cus.h"
#include <string.h>
#include "nrf_gpio.h"
#include "boards.h"
#include "nrf_log.h"
```

次のステップは、ble_cus.h のインクルードの下に次のスニペットを追加して、カスタム サービス (ble_cus) インスタンスを定義するマクロを追加することです。

```c
/* This code belongs in ble_cus.h*/

/**@brief   Macro for defining a ble_cus instance.
 *
 * @param   _name   Name of the instance.
    * @hideinitializer
 */
#define BLE_CUS_DEF(_name)                                                                          \
static ble_cus_t _name;                                                                             \

```

このマクロを使用して、チュートリアルの後半で main.c にカスタム サービス インスタンスを定義します。

わかりました、これまでのところとても良いです。次に、ble_cus.h に 2 つの構造体を作成する必要があります。1 つはカスタム サービスの初期化構造体、ble_cus_init_t 構造体は、カスタム サービスを初期化するために必要なすべてのオプションとデータを保持します。

```c
/* This code belongs in ble_cus.h*/

/**@brief Custom Service init structure. This contains all options and data needed for
 *        initialization of the service.*/
typedef struct
{
    uint8_t                       initial_custom_value;           /**< Initial custom value */
    ble_srv_cccd_security_mode_t  custom_value_char_attr_md;     /**< Initial security level for Custom characteristics attribute */
} ble_cus_init_t;
```

作成する必要がある 2 番目の構造体は、サービスのステータス情報を保持するカスタム サービス構造体 ble_cus_s です。

```c
/* This code belongs in ble_cus.h*/

/**@brief Custom Service structure. This contains various status information for the service. */
struct ble_cus_s
{
    uint16_t                      service_handle;                 /**< Handle of Custom Service (as provided by the BLE stack). */
    ble_gatts_char_handles_t      custom_value_handles;           /**< Handles related to the Custom Value characteristic. */
    uint16_t                      conn_handle;                    /**< Handle of the current connection (as provided by the BLE stack, is BLE_CONN_HANDLE_INVALID if not in a connection). */
    uint8_t                       uuid_type;
};
```

次のステップは、ble_cus_t タイプの前方宣言を追加することです。

```c
/* This code belongs in ble_cus.h*/

// Forward declaration of the ble_cus_t type.
typedef struct ble_cus_s ble_cus_t;
```

ble_cus_t がこのマクロで使用されているため、BLE_CUS_DEF マクロの上に、つまりこの順序でこれを追加してください。

```c
/* This code belongs in ble_cus.h*/

// Forward declaration of the ble_cus_t type.
typedef struct ble_cus_s ble_cus_t;

/**@brief   Macro for defining a ble_cus instance.
 *
 * @param   _name   Name of the instance.
    * @hideinitializer
 */
#define BLE_CUS_DEF(_name)                                                                          \
static ble_cus_t _name;                                                                             \

```

最初に実装する関数は ble_cus_init 関数で、これを使用してサービスを初期化します。まず、ble_cus.h ファイルに関数 decleration を追加する必要があります。

```c
/* This code belongs in ble_cus.h*/

/**@brief Function for initializing the Custom Service.
 *
 * @param[out]  p_cus       Custom Service structure. This structure will have to be supplied by
    *                          the application. It will be initialized by this function, and will later
    *                          be used to identify this particular service instance.
    * @param[in]   p_cus_init  Information needed to initialize the service.
 *
 * @return      NRF_SUCCESS on successful initialization of service, otherwise an error code.
 */
uint32_t ble_cus_init(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init);
```

ble_cus_init に入って最初にすべきことは、引数として渡したポインタが NULL でないことを確認し、2 つの変数 err_code と ble_uuid を宣言することです。

```c
/* This code belongs in ble_cus.c*/

uint32_t ble_cus_init(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init)
{
    if (p_cus == NULL || p_cus_init == NULL)
    {
        return NRF_ERROR_NULL;
    }

    uint32_t   err_code;
    ble_uuid_t ble_uuid;
}
```

ポインターが有効であることを確認した後、カスタム サービス構造を初期化できます。

```c
/* This code belongs in ble_cus_init() in ble_cus.c*/

// Initialize service structure
p_cus->conn_handle               = BLE_CONN_HANDLE_INVALID;
```

これは、接続ハンドルを無効に設定することで構成されます (接続中にのみ有効にする必要があります)。次に、カスタム (ベンダー固有とも呼ばれる) ベース UUID を BLE スタックのテーブルに追加します。

```c
/* This code belongs in ble_cus_init() ble_cus.c*/

// Add Custom Service UUID
ble_uuid128_t base_uuid = {CUSTOM_SERVICE_UUID_BASE};
err_code =  sd_ble_uuid_vs_add(&base_uuid, &p_cus->uuid_type);
VERIFY_SUCCESS(err_code);

ble_uuid.type = p_cus->uuid_type;
ble_uuid.uuid = CUSTOM_SERVICE_UUID;
```

ほぼ完了です。最後に行う必要があるのは、カスタム サービス宣言を BLE スタックの GATT テーブルに追加することです。

```c
/* This code belongs in ble_cus_init() in ble_cus.c*/

// Add the Custom Service
err_code = sd_ble_gatts_service_add(BLE_GATTS_SRVC_TYPE_PRIMARY, &ble_uuid, &p_cus->service_handle);
if (err_code != NRF_SUCCESS)
{
    return err_code;
}
```

手順を正しく実行した場合、ble_cus_init は次のようになります。

```c
/* This code belongs in ble_cus.c*/

uint32_t ble_cus_init(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init)
{
    if (p_cus == NULL || p_cus_init == NULL)
    {
        return NRF_ERROR_NULL;
    }

    uint32_t   err_code;
    ble_uuid_t ble_uuid;

    // Initialize service structure
    p_cus->conn_handle               = BLE_CONN_HANDLE_INVALID;

    // Add Custom Service UUID
    ble_uuid128_t base_uuid = {CUSTOM_SERVICE_UUID_BASE};
    err_code =  sd_ble_uuid_vs_add(&base_uuid, &p_cus->uuid_type);
    VERIFY_SUCCESS(err_code);
    
    ble_uuid.type = p_cus->uuid_type;
    ble_uuid.uuid = CUSTOM_SERVICE_UUID;

    // Add the Custom Service
    err_code = sd_ble_gatts_service_add(BLE_GATTS_SRVC_TYPE_PRIMARY, &ble_uuid, &p_cus->service_handle);
    VERIFY_SUCCESS(err_code);
    
    return err_code;
}
```

### ステップ 3 - サービスを初期化し、128 ビット UUID をアドバタイズします。

ここで、main.c でサービスを初期化し、アドバタイズ パケットに 128 ビット UUID を挿入して、他の BLE デバイスがデバイスにカスタム サービスがあることを認識できるようにします。

まず、ble_cus.h ファイルを main.c のインクルード リストに追加します。

```c
#include "ble_cus.h"
```

次に、BLE_CUS_DEF マクロを使用して、m_cus と呼ばれるカスタム サービス インスタンスを追加します。つまり、main.c の定義の下に次の行を追加します。

```c
BLE_CUS_DEF(m_cus);
```

次のステップは、main.c で空の services_init 関数を見つけることです。これは次のようになります。

```c
/**@brief Function for initializing services that will be used by the application.
 */
static void services_init(void)
{
    ret_code_t         err_code;
    nrf_ble_qwr_init_t qwr_init = {0};

    // Initialize Queued Write Module.
    qwr_init.error_handler = nrf_qwr_error_handler;

    err_code = nrf_ble_qwr_init(&m_qwr, &qwr_init);
    APP_ERROR_CHECK(err_code);

    /* YOUR_JOB: Add code to initialize the services used by the application.
       ble_xxs_init_t                     xxs_init;
       ble_yys_init_t                     yys_init;

       // Initialize XXX Service.
       memset(&xxs_init, 0, sizeof(xxs_init));

       xxs_init.evt_handler                = NULL;
       xxs_init.is_xxx_notify_supported    = true;
       xxs_init.ble_xx_initial_value.level = 100;

       err_code = ble_bas_init(&m_xxs, &xxs_init);
       APP_ERROR_CHECK(err_code);

       // Initialize YYY Service.
       memset(&yys_init, 0, sizeof(yys_init));
       yys_init.evt_handler                  = on_yys_evt;
       yys_init.ble_yy_initial_value.counter = 0;

       err_code = ble_yy_service_init(&yys_init, &yy_init);
       APP_ERROR_CHECK(err_code);
     */
}
```

では、言われたとおりに実行します。つまり、ble_cus_init_t 構造体を作成し、必要なデータを入力して、それを引数としてサービス初期化関数 ble_cus_init(); に渡します。

```c
/**@brief Function for initializing services that will be used by the application.
 */
 static void services_init(void)
{
    ret_code_t         err_code;
    nrf_ble_qwr_init_t qwr_init = {0};

    // Initialize Queued Write Module.
    qwr_init.error_handler = nrf_qwr_error_handler;

    err_code = nrf_ble_qwr_init(&m_qwr, &qwr_init);
    APP_ERROR_CHECK(err_code);
    
    ble_cus_init_t                     cus_init;

     // Initialize CUS Service init structure to zero.
    memset(&cus_init, 0, sizeof(cus_init));
	
    err_code = ble_cus_init(&m_cus, &cus_init);
    APP_ERROR_CHECK(err_code);

    /* YOUR_JOB: Add code to initialize the services used by the application.
       ble_xxs_init_t                     xxs_init;
       ble_yys_init_t                     yys_init;

       // Initialize XXX Service.
       memset(&xxs_init, 0, sizeof(xxs_init));

       xxs_init.evt_handler                = NULL;
       xxs_init.is_xxx_notify_supported    = true;
       xxs_init.ble_xx_initial_value.level = 100;

       err_code = ble_bas_init(&m_xxs, &xxs_init);
       APP_ERROR_CHECK(err_code);

       // Initialize YYY Service.
       memset(&yys_init, 0, sizeof(yys_init));
       yys_init.evt_handler                  = on_yys_evt;
       yys_init.ble_yy_initial_value.counter = 0;

       err_code = ble_yy_service_init(&yys_init, &yy_init);
       APP_ERROR_CHECK(err_code);
     */
}
```

サービスを初期化したので、アドバタイズ パケットに 128 ビット UUID を追加する必要があります。 main.c の先頭に移動すると、m_adv_uuids 配列が見つかるはずです。

```c
// YOUR_JOB: Use UUIDs for service(s) used in your application.
static ble_uuid_t m_adv_uuids[] = {{BLE_UUID_DEVICE_INFORMATION_SERVICE, BLE_UUID_TYPE_BLE}}; /**< Universally unique service identifiers. */
```

BLE_UUID_DEVICE_INFORMATION_SERVICE を ble_cus.h で定義した CUSTOM_SERVICE_UUID に置き換える必要があります。また、BLE_UUID_TYPE_BLE を BLE_UUID_TYPE_VENDOR_BEGIN に置き換える必要があります。これは、128 ビットのベンダー固有の UUID であり、16 ビットの Bluetooth SIG UUDID ではないためです。 m_adv_uuids は次のようになります。

```c
// YOUR_JOB: Use UUIDs for service(s) used in your application.
 ble_uuid_t m_adv_uuids[] = {{CUSTOM_SERVICE_UUID, BLE_UUID_TYPE_VENDOR_BEGIN }}; /**< Universally unique service identifiers. */
```

このステップの後、16 ビットの UUID ではなく、ベンダー固有の 128 ビットの UUID を使用していることを BLE スタックに伝える必要があります。これは、sdk_config.h の次の定義を

```c
#define NRF_SDH_BLE_VS_UUID_COUNT 0
```

に

```
#define NRF_SDH_BLE_VS_UUID_COUNT 1
```

現在、ベンダー固有の UUID を BLE スタックに追加すると、SoftDevice の RAM 要件が増加するため、これを考慮する必要があります。

<!---
- [ ] Optional: Add section where the function of app_ram_base since its useful for debugging.
--->

**GCC:** armgcc を使用してコードをコピーしている場合は、nRF5_SDK\nRF5_SDK_15.0.0_a53641a\examples\ble_peripheral\custom_ble_service_example\pca10040\g1 フォルダーにある ble_app_template_gcc_nrf52.ld ファイルを開き、Memory セクションの下にあるように変更する必要があります。 .

```c
MEMORY
{
  FLASH (rx) : ORIGIN = 0x26000, LENGTH = 0x5a000
  RAM (rwx) :  ORIGIN = 0x20002220, LENGTH = 0xdde0
}
```

**Segger Embedded Studio (SES):** [プロジェクト] -&gt; [オプションの編集] をクリックし、[共通設定] を選択し、[リンカ] を選択して [セクション配置マクロ] セクションを開き、次のスクリーンショットに示すように RAM_START IRAM1 を 0x20002220 に、RAM_SIZE を 0xDDE0 に変更します。

メモリ設定 Segger Embedded Studio
---
<img src="https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/images/memory_settings_SDK_v15_SES.JPG" width="1000">

**Keil: Keil の**[Options for Target] をクリックし、以下のスクリーンショットに示すように、IRAM1 の開始アドレスが 0x20002220 でサイズが 0xDDE0 になるように、読み取り/書き込みメモリ領域を変更します。

メモリー設定 Keil
---
<img src="https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/images/memory_settings_SDK_v15_Keil.JPG" width="1000">

しなければならない最後のステップは、services_init() が Advertising_init() の前に呼び出されるように、main() の呼び出し順序を変更することです。これは、advertising_init() を呼び出す前に、ble_cus_init() の sd_ble_uuid_vs_add() を使用して、CUSTOM_SERVICE_UUID_BASE を BLE スタックのテーブルに追加する必要があるためです。逆に実行すると、advertising_init() はエラー コードを返します。

これで、ble_app_template プロジェクトをコンパイルし、S132 v6.0.0 SoftDevice をフラッシュしてから、ble_app_template アプリケーションをフラッシュします (Keil を使用している場合にのみ適用されます)。 nRF52 DK の LED1 が点滅を開始し、アドバタイズしていることを示します。 nRF Connect for Android/iOS を使用してデバイスをスキャンし、広告パッケージのコンテンツを表示します。デバイスに接続すると、ベンダー固有の UUID を使用しているため、サービスが「不明なサービス」としてリストされていることがわかります。

広告装置 | 広告パケットの内容 | GATT表に掲載されているサービス
--- | --- | ---
<img src="https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/images/advertising_device.png" width="250"> | <img src="https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/images/advertisement_packet.png" width="250 "> | <img src="https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/images/service.png" width="250">

### ステップ 4 - カスタム サービスにカスタム値特性を追加します。

サービスは特性のないものではありません。したがって、ble_cus.c に custom_value_char_add 関数を作成して、サービスの 1 つを追加してみましょう。

最初に行う必要があるのは、関数を宣言してから、以下のスニペットに示すように、後で入力するいくつかのメタデータ変数を追加することです。

```c
/* This code belongs in ble_cus.c*/

/**@brief Function for adding the Custom Value characteristic.
 *
 * @param[in]   p_cus        Custom Service structure.
    * @param[in]   p_cus_init   Information needed to initialize the service.
 *
 * @return      NRF_SUCCESS on success, otherwise an error code.
 */
static uint32_t custom_value_char_add(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init)
{
    uint32_t            err_code;
    ble_gatts_char_md_t char_md;
    ble_gatts_attr_md_t cccd_md;
    ble_gatts_attr_t    attr_char_value;
    ble_uuid_t          ble_uuid;
    ble_gatts_attr_md_t attr_md;
}
```

ここで、これらすべての変数を設定するというかなり退屈な部分を開始します。char_md から始めます。これは、サービス検出中に中央に表示されるプロパティを設定します。

```c
/* This code belongs in custom_value_char_add() in ble_cus.c*/

    memset(&char_md, 0, sizeof(char_md));

    char_md.char_props.read   = 1;
    char_md.char_props.write  = 1;
    char_md.char_props.notify = 0;
    char_md.p_char_user_desc  = NULL;
    char_md.p_char_pf         = NULL;
    char_md.p_user_desc_md    = NULL;
    char_md.p_cccd_md         = NULL;
    char_md.p_sccd_md         = NULL;
}
```

そのため、カスタム値特性に対して書き込みと読み取りの両方を行えるようにしたいのですが、通知プロパティは後で有効にしたくありません。次に、実際にプロパティ (つまり、属性のアクセス可能性) を設定する attr_md を設定します。

```c
    /* This code belongs in custom_value_char_add() in ble_cus.c*/

    memset(&attr_md, 0, sizeof(attr_md));

    attr_md.read_perm  = p_cus_init->custom_value_char_attr_md.read_perm;
    attr_md.write_perm = p_cus_init->custom_value_char_attr_md.write_perm;
    attr_md.vloc       = BLE_GATTS_VLOC_STACK;
    attr_md.rd_auth    = 0;
    attr_md.wr_auth    = 0;
    attr_md.vlen       = 0;
}
```

attr_md 構造体で設定された権限は、特性メタデータ構造体 char_md で設定されたプロパティに対応する必要があります。 services_init() の ble_cus_init に渡すカスタム サービスの init 構造でアクセス許可を提供します。特性をアプリケーション RAM セクションではなく SoftDevice RAM セクションに保存するため、.vloc オプションは BLE_GATTS_VLOC_STACK に設定されています。

次に入力する必要がある変数は ble_uuid です。これは、CUSTOM_VALUE_CHAR_UUID を保持し、CUSTOM_SERVICE_UUID_BASE と同じタイプ (つまり、ベンダー固有) です。これは、CUSTOM_SERVICE_UUID_BASE をBLE スタックのテーブル。

```c
    /* This code belongs in custom_value_char_add() in ble_cus.c*/

    ble_uuid.type = p_cus->uuid_type;
    ble_uuid.uuid = CUSTOM_VALUE_CHAR_UUID;
```

次に、UUID を設定し、属性メタデータを指し、特性のサイズを設定する attr_char_value 構造体を設定します。この場合は、1 バイト (uint8_t) です。

```c
    /* This code belongs in custom_value_char_add() in ble_cus.c*/

    memset(&attr_char_value, 0, sizeof(attr_char_value));

    attr_char_value.p_uuid    = &ble_uuid;
    attr_char_value.p_attr_md = &attr_md;
    attr_char_value.init_len  = sizeof(uint8_t);
    attr_char_value.init_offs = 0;
    attr_char_value.max_len   = sizeof(uint8_t);
```

最後に、構造体の設定が完了し、構造体を引数として sd_ble_gatts_characteristic_add() を呼び出すことで特性を追加できます。

```c
/* This code belongs in custom_value_char_add() in ble_cus.c*/

err_code = sd_ble_gatts_characteristic_add(p_cus->service_handle, &char_md,
                                               &attr_char_value,
                                               &p_cus->custom_value_handles);
    if (err_code != NRF_SUCCESS)
    {
        return err_code;
    }

    return NRF_SUCCESS;
```

そのすべてのハードワーク (コピーと貼り付け) の後、custom_value_char_add() 関数は次のようになります。

```c
/* This code belongs in ble_cus.c*/

static uint32_t custom_value_char_add(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init)
{
    uint32_t            err_code;
    ble_gatts_char_md_t char_md;
    ble_gatts_attr_md_t cccd_md;
    ble_gatts_attr_t    attr_char_value;
    ble_uuid_t          ble_uuid;
    ble_gatts_attr_md_t attr_md;

    memset(&char_md, 0, sizeof(char_md));

    char_md.char_props.read   = 1;
    char_md.char_props.write  = 1;
    char_md.char_props.notify = 0;
    char_md.p_char_user_desc  = NULL;
    char_md.p_char_pf         = NULL;
    char_md.p_user_desc_md    = NULL;
    char_md.p_cccd_md         = NULL;
    char_md.p_sccd_md         = NULL;
		
    memset(&attr_md, 0, sizeof(attr_md));

    attr_md.read_perm  = p_cus_init->custom_value_char_attr_md.read_perm;
    attr_md.write_perm = p_cus_init->custom_value_char_attr_md.write_perm;
    attr_md.vloc       = BLE_GATTS_VLOC_STACK;
    attr_md.rd_auth    = 0;
    attr_md.wr_auth    = 0;
    attr_md.vlen       = 0;

    ble_uuid.type = p_cus->uuid_type;
    ble_uuid.uuid = CUSTOM_VALUE_CHAR_UUID;

    memset(&attr_char_value, 0, sizeof(attr_char_value));

    attr_char_value.p_uuid    = &ble_uuid;
    attr_char_value.p_attr_md = &attr_md;
    attr_char_value.init_len  = sizeof(uint8_t);
    attr_char_value.init_offs = 0;
    attr_char_value.max_len   = sizeof(uint8_t);

    err_code = sd_ble_gatts_characteristic_add(p_cus->service_handle, &char_md,
                                               &attr_char_value,
                                               &p_cus->custom_value_handles);
    if (err_code != NRF_SUCCESS)
    {
        return err_code;
    }

    return NRF_SUCCESS;
}
```

最後のステップは、サービスが追加された ble_cus_init の最後に custom_value_char_add() を呼び出すことです。つまり、

```c
/* This code belongs in ble_cus.c*/

uint32_t ble_cus_init(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init)
{
    if (p_cus == NULL || p_cus_init == NULL)
    {
        return NRF_ERROR_NULL;
    }

    uint32_t   err_code;
    ble_uuid_t ble_uuid;

    // Initialize service structure
    p_cus->conn_handle               = BLE_CONN_HANDLE_INVALID;

    // Add Custom Service UUID
    ble_uuid128_t base_uuid = {CUSTOM_SERVICE_UUID_BASE};
    err_code =  sd_ble_uuid_vs_add(&base_uuid, &p_cus->uuid_type);
    VERIFY_SUCCESS(err_code);
    
    ble_uuid.type = p_cus->uuid_type;
    ble_uuid.uuid = CUSTOM_SERVICE_UUID;

    // Add the Custom Service
    err_code = sd_ble_gatts_service_add(BLE_GATTS_SRVC_TYPE_PRIMARY, &ble_uuid, &p_cus->service_handle);
    if (err_code != NRF_SUCCESS)
    {
        return err_code;
    }

    // Add Custom Value characteristic
    return custom_value_char_add(p_cus, p_cus_init);
}
```

プロジェクトをコンパイルし、nRF5x DK にフラッシュします。スマートフォンで nRF Connect アプリを開き、デバイスをスキャンして接続すると、下のスクリーンショットに示すように、サービスをクリックして特性が追加されていることがわかります。

サービスと特徴
---
<img src="https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/images/service_and_char.png" width="250">

### ステップ 5 - SoftDevice からのイベントの処理。

これでカスタム サービスとカスタム値の特性ができましたが、特性に書き込んで、特性に書き込まれた値に基づいて特定のタスクを実行できるようにしたいと思います。たとえば、LED をオンにします。ただし、その前に、ble_cus.h と ble_cus.c でイベント処理を行う必要があります。

最後に、ble_cus_on_ble_evt 関数 decleration を ble_cus.h に追加します。これは、サービスからの ble_cus_evt_type_t のイベントを処理します。

```c
/* This code belongs in ble_cus.h*/

/**@brief Function for handling the Application's BLE Stack events.
 *
 * @details Handles all events from the BLE stack of interest to the Battery Service.
 *
 * @note
 *
 * @param[in]   p_ble_evt  Event received from the BLE stack.
    * @param[in]   p_context  Custom Service structure.
 */
void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context);
```

ble_cus_on_ble_evt イベント ハンドラーを ble_cus.c に実装します。エントリで、引数として提供したポインターが NULL でないことを確認することをお勧めします。

```c
/* This code belongs in ble_cus.c*/

void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context)
{
    ble_cus_t * p_cus = (ble_cus_t *) p_context;
    
    if (p_cus == NULL || p_ble_evt == NULL)
    {
        return;
    }
}
```

NULL チェックの後、SoftDevice によってアプリケーションに伝達されたイベントをチェックするスイッチ ケースを追加します。

```c
/* This code belongs in ble_cus.c*/

void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context)
{
    ble_cus_t * p_cus = (ble_cus_t *) p_context;
    
    if (p_cus == NULL || p_ble_evt == NULL)
    {
        return;
    }
    
    switch (p_ble_evt->header.evt_id)
    {
        case BLE_GAP_EVT_CONNECTED:
            break;

        case BLE_GAP_EVT_DISCONNECTED:
            break;

        default:
            // No implementation needed.
            break;
    }
}
```

現時点では、BLE_GAP_EVT_CONNECTED および BLE_GAP_EVT_DISCONNECTED イベントのみを考慮する必要があります。それでは、on_connect() から始めて、2 つの関数 on_connect() と on_disconnect() を作成しましょう。 Connect イベントを取得するときに必要なことは、カスタム サービス構造の接続ハンドルを、イベントで渡される接続ハンドルに割り当てることだけです。

```c
/* This code belongs in ble_cus.c*/

/**@brief Function for handling the Connect event.
 *
 * @param[in]   p_cus       Custom Service structure.
    * @param[in]   p_ble_evt   Event received from the BLE stack.
 */
static void on_connect(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    p_cus->conn_handle = p_ble_evt->evt.gap_evt.conn_handle;
}
```

同様に、Disconnect イベントを取得すると、接続が切断されたため、カスタム サービス構造の接続ハンドルを無効にするだけで済みます。

```c
/* This code belongs in ble_cus.c*/

/**@brief Function for handling the Disconnect event.
 *
 * @param[in]   p_cus       Custom Service structure.
    * @param[in]   p_ble_evt   Event received from the BLE stack.
 */
static void on_disconnect(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    UNUSED_PARAMETER(p_ble_evt);
    p_cus->conn_handle = BLE_CONN_HANDLE_INVALID;
}
```

イベントごとに 1 つの関数ができたので、ble_cus_on_ble_evt の関数を呼び出すだけです。つまり、

```c
/* This code belongs in ble_cus.c*/

void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context)
{
    ble_cus_t * p_cus = (ble_cus_t *) p_context;

    if (p_cus == NULL || p_ble_evt == NULL)
    {
        return;
    }
    
    switch (p_ble_evt->header.evt_id)
    {
        case BLE_GAP_EVT_CONNECTED:
            on_connect(p_cus, p_ble_evt);
            break;

        case BLE_GAP_EVT_DISCONNECTED:
            on_disconnect(p_cus, p_ble_evt);
            break;

        default:
            // No implementation needed.
            break;
    }
}
```

最後に、ble_cus_on_ble_evt イベント ハンドラー関数が SoftDevice イベントを受信するようにする必要があります。これは、NRF_SDH_BLE_OBSERVER() マクロを使用して、ble_cus_on_ble_evt イベント ハンドラーをイベント オブザーバーとして登録して行われます。 ble_cus.h で定義した BLE_CUS_DEF マクロ内でこれを行うと便利です。これは、以下に示すように変更する必要があります。

```c
/* This code belongs in ble_cus.h*/

#define BLE_CUS_DEF(_name)                                                                          \
static ble_cus_t _name;                                                                             \
NRF_SDH_BLE_OBSERVER(_name ## _obs,                                                                 \
                     BLE_HRS_BLE_OBSERVER_PRIO,                                                     \
                     ble_cus_on_ble_evt, &_name)

```

次のステップに進む前に、プロジェクトをコンパイルし、エラーがないことを確認してください。

### ステップ 6 - SoftDevice からの Write イベントの処理。

さて、これで、特性に書き込み、特性に書き込まれた値に基づいて特定のタスクを実行できるようになりたいと考えています。どうやって行くの？当たってるよ！イベント処理がさらに充実！

特性が書き込まれるたびに、BLE_GATTS_EVT_WRITE イベントがアプリケーションに伝達され、ble_evt_dispatch() の関数にディスパッチされます。つまり、これは、ble_cus_on_ble_evt switch-case ステートメントに別のケース、つまり BLE_GATTS_EVT_WRITE を追加する必要があることを意味します。

```c
/* This code belongs in ble_cus.c*/

void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context)
{
   ble_cus_t * p_cus = (ble_cus_t *) p_context;

   if (p_cus == NULL || p_ble_evt == NULL)
   {
       return;
   }
   
   switch (p_ble_evt->header.evt_id)
   {
       case BLE_GAP_EVT_CONNECTED:
           on_connect(p_cus, p_ble_evt);
           break;

       case BLE_GAP_EVT_DISCONNECTED:
           on_disconnect(p_cus, p_ble_evt);
           break;
       case BLE_GATTS_EVT_WRITE:
           break;
       default:
           // No implementation needed.
           break;
   }
}
```

BLE_GAP_EVT_CONNECTED と BLE_GAP_EVT_DISCONNECTED の場合と同じように、Write イベントを取得したときに呼び出される on_write 関数を作成します。

```c
/* This code belongs in ble_cus.c*/

/**@brief Function for handling the Write event.
 *
 * @param[in]   p_cus       Custom Service structure.
    * @param[in]   p_ble_evt   Event received from the BLE stack.
 */
static void on_write(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{

}
```

書き込みイベントを取得したら、イベントで渡される書き込みイベント パラメーターを取得し、書き込まれるハンドルがカスタム値の特性ハンドルと一致することを確認する必要があります。

```c
/* This code belongs in ble_cus.c*/

static void on_write(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    ble_gatts_evt_write_t * p_evt_write = &p_ble_evt->evt.gatts_evt.params.write;
    
    // Check if the handle passed with the event matches the Custom Value Characteristic handle.
    if (p_evt_write->handle == p_cus->custom_value_handles.value_handle)
    {
        // Put specific task here.
    }
}
```

したがって、私たちの特定のタスクは、カスタム値の特性が書き込まれるたびに nRF5x DK の LED を切り替えることであるとします。これを行うには、nRF5x DK LED に接続されているピンの 1 つ、たとえば LED4 で nrf_gpio_pin_toggle を呼び出します。これを行うには、boards.h と nrf_gpio.h を ble_cus.h にインクルードし、on_write 関数で nrf_gpio_pin_toggle を呼び出す必要があります。

```c
/* This code belongs in ble_cus.c*/

static void on_write(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    ble_gatts_evt_write_t * p_evt_write = &p_ble_evt->evt.gatts_evt.params.write;
    
    // Check if the handle passed with the event matches the Custom Value Characteristic handle.
    if (p_evt_write->handle == p_cus->custom_value_handles.value_handle)
    {
        nrf_gpio_pin_toggle(LED_4);
    }
}
```

必要なイベント処理を実装したので、BLE_GATTS_EVT_WRITE ケースで on_write() を ble_cus_on_ble_evt() 関数に追加する必要があります。

```c
/* This code belongs in ble_cus.c*/

void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context)
{
   ble_cus_t * p_cus = (ble_cus_t *) p_context;

   if (p_cus == NULL || p_ble_evt == NULL)
   {
       return;
   }
   
   switch (p_ble_evt->header.evt_id)
   {
       case BLE_GAP_EVT_CONNECTED:
           on_connect(p_cus, p_ble_evt);
           break;

       case BLE_GAP_EVT_DISCONNECTED:
           on_disconnect(p_cus, p_ble_evt);
           break;
       case BLE_GATTS_EVT_WRITE:
           on_write(p_cus, p_ble_evt);
           break;
       default:
           // No implementation needed.
           break;
   }
}
```

ただし、ピアが実際に特性値を読み書きすることを許可しない場合、これはすべて無駄になります。これは、ble_cus_init() を呼び出す前に、main.c の services_init() に 2 行を追加することによって行われます。

```c
/* This code belongs in services_init() in main.c*/

BLE_GAP_CONN_SEC_MODE_SET_OPEN(&cus_init.custom_value_char_attr_md.read_perm);
BLE_GAP_CONN_SEC_MODE_SET_OPEN(&cus_init.custom_value_char_attr_md.write_perm);
```

これらの 2 行は、書き込みと読み取りのアクセス許可を特性値属性に設定します。つまり、ピアは最初にリンクを暗号化せずに値の書き込み/読み取りを許可されます。ここで、nRF Connect for Desktop または Android/iOS を使用して、キャラクタリスティックへの書き込みを試みます。特性に値が書き込まれるたびに、nRF5x DK の LED4 が点灯します。

書き込みボタン | 値を書き込む
--- | ---
<img src="https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/images/write_to_char_1.png" width="250"> | <img src="https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/images/write_to_char_2.png" width="250">

**課題 1:** p_evt_write にもデータ フィールドがあります。データを使用して、LED をオンにするかオフにするかを決定します。

### ステップ 7 - カスタム サービス イベントをアプリケーションに伝達する

これまで、SoftDevice によって伝達されるイベントのみを処理してきましたが、場合によっては、イベントをアプリケーションに伝達することが理にかなっています。

これを行うには、カスタム サービス初期化構造とカスタム サービス構造にイベント ハンドラーを追加する必要があります。

```c
 /* This code belongs in ble_cus.h*/

/**@brief Custom Service init structure. This contains all options and data needed for
 *        initialization of the service.*/
typedef struct
{
    ble_cus_evt_handler_t         evt_handler;                    /**< Event handler to be called for handling events in the Custom Service. */
    uint8_t                       initial_custom_value;           /**< Initial custom value */
    ble_srv_cccd_security_mode_t  custom_value_char_attr_md;     /**< Initial security level for Custom characteristics attribute */
} ble_cus_init_t;

/**@brief Custom Service structure. This contains various status information for the service. */
struct ble_cus_s
{
    ble_cus_evt_handler_t         evt_handler;                    /**< Event handler to be called for handling events in the Custom Service. */
    uint16_t                      service_handle;                 /**< Handle of Custom Service (as provided by the BLE stack). */
    ble_gatts_char_handles_t      custom_value_handles;           /**< Handles related to the Custom Value characteristic. */
    uint16_t                      conn_handle;                    /**< Handle of the current connection (as provided by the BLE stack, is BLE_CONN_HANDLE_INVALID if not in a connection). */
    uint8_t                       uuid_type;
};
```

構造にイベント ハンドラーを追加した後、ble_cus_init() でサービスを正しく初期化する必要があります。

```c
 /* This code belongs in ble_cus_init() in ble_cus.c*/

// Initialize service structure
p_cus->evt_handler               = p_cus_init->evt_handler;
p_cus->conn_handle               = BLE_CONN_HANDLE_INVALID;
```

次に、サービスに固有のイベント タイプを宣言する必要があります。

```c
 /* This code belongs in ble_cus.h*/

typedef enum
{
    BLE_CUS_EVT_DISCONNECTED,
    BLE_CUS_EVT_CONNECTED
} ble_cus_evt_type_t;
```

現時点では、BLE_CUS_EVT_CONNECTED および BLE_CUS_EVT_DISCONNECTED イベントのみを追加しますが、チュートリアルの後半でいくつかのイベントを追加します。

イベント タイプを宣言した後、ble_cus_evt_type_t イベントを保持するイベント構造を宣言する必要があります。つまり、

```c
 /* This code belongs in ble_cus.h*/

/**@brief Custom Service event. */
typedef struct
{
    ble_cus_evt_type_t evt_type;                                  /**< Type of event. */
} ble_cus_evt_t;
```

次に、カスタム サービス イベント ハンドラー タイプを宣言する必要があります。

```c
 /* This code belongs in ble_cus.h*/

/**@brief Custom Service event handler type. */
typedef void (*ble_cus_evt_handler_t) (ble_cus_t * p_cus, ble_cus_evt_t * p_evt);
```

ここで、メインに戻って、ble_cus_evt_handler_t タイプと同じパラメーターを受け取るイベント ハンドラー関数 on_cus_evt を作成します。

```c
 /* This code belongs in main.c*/

/**@brief Function for handling the Custom Service Service events.
 *
 * @details This function will be called for all Custom Service events which are passed to
    *          the application.
 *
 * @param[in]   p_cus_service  Custom Service structure.
    * @param[in]   p_evt          Event received from the Custom Service.
 *
 */
static void on_cus_evt(ble_cus_t     * p_cus_service,
                       ble_cus_evt_t * p_evt)
{
    switch(p_evt->evt_type)
    {
        case BLE_CUS_EVT_CONNECTED:
            break;

        case BLE_CUS_EVT_DISCONNECTED:
              break;

        default:
              // No implementation needed.
              break;
    }
}
```

ここで、サービスからイベントを伝播するために、カスタム サービスを初期化するときに、サービスのイベント ハンドラー関数として on_cus_evt 関数を割り当てる必要があります。これは、main.c の services_init() で cus_init 構造体の .evt_handler フィールドを on_cus_evt に設定することによって行われます。

```c
 /* This code belongs in services_init in main.c*/

    // Set the cus event handler
    cus_init.evt_handler                = on_cus_evt;
```

p_cus-&gt;evt_handler(p_cus, &amp;evt) を呼び出すことにより、ble_cus.c からこのイベント ハンドラーを呼び出すことができます。例として、BLE_GAP_EVT_CONNECTED イベントを取得したときに、つまり on_connect() 関数でイベント ハンドラーを呼び出します。かなり簡単です。ble_cus_evt_t 変数を宣言し、その .evt_type フィールドを BLE_CUS_EVT_CONNECTED に設定してから、そのイベントでイベント ハンドラーを呼び出します。

```c
 /* This code belongs in ble_cus.c*/

static void on_connect(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    p_cus->conn_handle = p_ble_evt->evt.gap_evt.conn_handle;

    ble_cus_evt_t evt;

    evt.evt_type = BLE_CUS_EVT_CONNECTED;

    p_cus->evt_handler(p_cus, &evt);
}
```

### ステップ 8 - カスタム値特性の通知

次に、ble_cus_custom_value_update 関数 decleration を ble_cus.h ファイルに追加します。これを使用して、カスタム値特性を更新します。

```c
 /* This code belongs in ble_cus.h*/

/**@brief Function for updating the custom value.
 *
 * @details The application calls this function when the cutom value should be updated. If
    *          notification has been enabled, the custom value characteristic is sent to the client.
 *
 * @note
    *
    * @param[in]   p_cus          Custom Service structure.
    * @param[in]   Custom value
 *
 * @return      NRF_SUCCESS on success, otherwise an error code.
 */

uint32_t ble_cus_custom_value_update(ble_cus_t * p_cus, uint8_t custom_value);
```

ble_cus.c に戻って、引数として渡したポインターが NULL でないことを確認する良い習慣を続けます。

```c
 /* This code belongs in ble_cus.c*/

uint32_t ble_cus_custom_value_update(ble_cus_t * p_cus, uint8_t custom_value){
    if (p_cus == NULL)
    {
        return NRF_ERROR_NULL;
    }
}
```

次に、ble_cus_custom_value_update() 関数に引数として渡した custom_value を使用して、GATT テーブルの値を更新します。

```c
/* This code belongs in ble_cus_custom_value_update() in ble_cus.c*/

uint32_t err_code = NRF_SUCCESS;
ble_gatts_value_t gatts_value;

// Initialize value struct.
memset(&gatts_value, 0, sizeof(gatts_value));

gatts_value.len     = sizeof(uint8_t);
gatts_value.offset  = 0;
gatts_value.p_value = &custom_value;

// Update database.
err_code = sd_ble_gatts_value_set(p_cus->conn_handle,
                                    p_cus->custom_value_handles.value_handle,
                                    &gatts_value);
if (err_code != NRF_SUCCESS)
{
    return err_code;
}
```

GATT テーブルの値を更新した後、カスタム値特性の値が変更されたことをピアに通知する準備ができました。

```c
/* This code belongs in ble_cus_custom_value_update() in ble_cus.c*/

// Send value if connected and notifying.
if ((p_cus->conn_handle != BLE_CONN_HANDLE_INVALID))
{
    ble_gatts_hvx_params_t hvx_params;

    memset(&hvx_params, 0, sizeof(hvx_params));

    hvx_params.handle = p_cus->custom_value_handles.value_handle;
    hvx_params.type   = BLE_GATT_HVX_NOTIFICATION;
    hvx_params.offset = gatts_value.offset;
    hvx_params.p_len  = &gatts_value.len;
    hvx_params.p_data = gatts_value.p_value;

    err_code = sd_ble_gatts_hvx(p_cus->conn_handle, &hvx_params);
}
else
{
    err_code = NRF_ERROR_INVALID_STATE;
}

return err_code;

```

最初に行うべきことは、有効な接続ハンドルがあること、つまり実際にピアに接続していることを確認することです。そうでない場合は、無効な状態であることを示すエラーを返す必要があります。次に、ble_gatts_hvx_params_t を設定します。これには、カスタム値属性のハンドル、タイプ (つまり、通知または表示の場合)、値のオフセット (20 バイトより大きい場合にのみ使用)、長さ、およびデータが含まれます。 hvx_params を設定した後、sd_ble_gatts_hvx() を呼び出してピアに通知します。最後に、エラー コードを返す必要があります。

ble_cus_custom_value_update() の上にコード スニペットを追加すると、以下のようになります。

```c
/* This code belongs in ble_cus.c*/

uint32_t ble_cus_custom_value_update(ble_cus_t * p_cus, uint8_t custom_value)
{
    NRF_LOG_INFO("In ble_cus_custom_value_update. \r\n");
    if (p_cus == NULL)
    {
        return NRF_ERROR_NULL;
    }

    uint32_t err_code = NRF_SUCCESS;
    ble_gatts_value_t gatts_value;

    // Initialize value struct.
    memset(&gatts_value, 0, sizeof(gatts_value));

    gatts_value.len     = sizeof(uint8_t);
    gatts_value.offset  = 0;
    gatts_value.p_value = &custom_value;

    // Update database.
    err_code = sd_ble_gatts_value_set(p_cus->conn_handle,
                                      p_cus->custom_value_handles.value_handle,
                                      &gatts_value);
    if (err_code != NRF_SUCCESS)
    {
        return err_code;
    }

    // Send value if connected and notifying.
    if ((p_cus->conn_handle != BLE_CONN_HANDLE_INVALID))
    {
        ble_gatts_hvx_params_t hvx_params;

        memset(&hvx_params, 0, sizeof(hvx_params));

        hvx_params.handle = p_cus->custom_value_handles.value_handle;
        hvx_params.type   = BLE_GATT_HVX_NOTIFICATION;
        hvx_params.offset = gatts_value.offset;
        hvx_params.p_len  = &gatts_value.len;
        hvx_params.p_data = gatts_value.p_value;

        err_code = sd_ble_gatts_hvx(p_cus->conn_handle, &hvx_params);
    }
    else
    {
        err_code = NRF_ERROR_INVALID_STATE;
    }


    return err_code;
}
```

特性メタデータと同様に、CCCD のメタデータを custom_value_char_add() に設定する必要があります。これは、特性メタデータが設定される前に、次のスニペットを追加することによって行われます。

```c
/* This code belongs in custom_value_char_add() in ble_cus.c*/

    memset(&cccd_md, 0, sizeof(cccd_md));

    //  Read  operation on Cccd should be possible without authentication.
    BLE_GAP_CONN_SEC_MODE_SET_OPEN(&cccd_md.read_perm);
    BLE_GAP_CONN_SEC_MODE_SET_OPEN(&cccd_md.write_perm);
    
    cccd_md.vloc       = BLE_GATTS_VLOC_STACK;
```

CCCD への読み取りおよび書き込み許可をオープンに設定しています。つまり、CCCD への書き込みまたは読み取りに暗号化は必要ありません。 .vloc オプションは BLE_GATTS_VLOC_STACK に設定されています。これは、CCCD の値がアプリケーション メモリ セクションではなく SoftDevice メモリ セクションに保存されることを意味します。

次の手順では、通知プロパティをカスタム値特性に追加し、特性メタデータに CCCD メタデータへのポインターを追加します。これは、custom_value_char_add() 関数の特性メタデータ プロパティを変更することによって行われます。つまり、.notify を 1 に設定し、.p_cccd_md を CCCD メタデータ構造体を指すように設定します。

```c
/* This code belongs in custom_value_char_add() in ble_cus.c*/
    char_md.char_props.notify = 1;
    char_md.p_cccd_md         = &cccd_md;
```

これにより、クライアント特性構成記述子または CCCD がカスタム値特性に追加され、CCCD に書き込むことで通知を有効または無効にできます。通知はデフォルトで無効になっており、有効にするには CCCD に 0x0001 を書き込む必要があります。キャラクタリスティックまたはその記述子の 1 つに書き込むたびに、Write イベントが発生することに注意してください。したがって、on_write() 関数でピアが CCCD に書き込む場合を処理する必要があります。

ただし、on_write() 関数を変更する前に、BLE_CUS_EVT_NOTIFICATION_ENABLED および BLE_CUS_EVT_NOTIFICATION_DISABLED という 2 つのイベントを ble_cus.h の ble_cus_evt_type_t 列挙に追加する必要があります。

```c
/* This code belongs in ble_cus.h*/

/**@brief Custom Service event type. */
typedef enum
{
    BLE_CUS_EVT_NOTIFICATION_ENABLED,                             /**< Custom value notification enabled event. */
    BLE_CUS_EVT_NOTIFICATION_DISABLED,                            /**< Custom value notification disabled event. */
    BLE_CUS_EVT_DISCONNECTED,
    BLE_CUS_EVT_CONNECTED
} ble_cus_evt_type_t;
```

on_write() 関数に戻ると、最初に行うべきことは、書き込まれるハンドルが CCCD のハンドルと一致し、値が正しい長さを持っているかどうかをチェックする if ステートメントを追加することです。このチェックに合格したら、通知が有効になっているかどうかを確認する必要があります。

```c
/* This code belongs in on_write() in ble_cus.c*/

    // Check if the Custom value CCCD is written to and that the value is the appropriate length, i.e 2 bytes.
    if ((p_evt_write->handle == p_cus->custom_value_handles.cccd_handle)
        && (p_evt_write->len == 2)
       )
    {

        // CCCD written, call application event handler
        if (p_cus->evt_handler != NULL)
        {
            ble_cus_evt_t evt;

            if (ble_srv_is_notification_enabled(p_evt_write->data))
            {
                evt.evt_type = BLE_CUS_EVT_NOTIFICATION_ENABLED;
            }
            else
            {
                evt.evt_type = BLE_CUS_EVT_NOTIFICATION_DISABLED;
            }
            // Call the application event handler.
            p_cus->evt_handler(p_cus, &evt);
        }

    }
```

これは SoftDevice によって自動的に行われるため、実際に通知を有効にする必要はありません。ただし、SoftDevice は書き込みイベントを伝播し、このイベントに基づいて、特性値の通知を開始してもよいかどうかを判断できます。

上記のコード スニペットを on_write() 関数に追加すると、次の関数が生成されます。

```c
/* This code belongs in ble_cus.c*/

static void on_write(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    ble_gatts_evt_write_t * p_evt_write = &p_ble_evt->evt.gatts_evt.params.write;
    
    // Custom Value Characteristic Written to.
    if (p_evt_write->handle == p_cus->custom_value_handles.value_handle)
    {
        nrf_gpio_pin_toggle(LED_4);
    }

    // Check if the Custom value CCCD is written to and that the value is the appropriate length, i.e 2 bytes.
    if ((p_evt_write->handle == p_cus->custom_value_handles.cccd_handle)
        && (p_evt_write->len == 2)
       )
    {
        // CCCD written, call application event handler
        if (p_cus->evt_handler != NULL)
        {
            ble_cus_evt_t evt;

            if (ble_srv_is_notification_enabled(p_evt_write->data))
            {
                evt.evt_type = BLE_CUS_EVT_NOTIFICATION_ENABLED;
            }
            else
            {
                evt.evt_type = BLE_CUS_EVT_NOTIFICATION_DISABLED;
            }
            // Call the application event handler.
            p_cus->evt_handler(p_cus, &evt);
        }
    }
}
```

最後に、BLE_CUS_EVT_NOTIFICATION_ENABLED と BLE_CUS_EVT_NOTIFICATION_DISABLED を main.c の on_cus_evt() イベント ハンドラに追加します。つまり、

```c
/* This code belongs in main.c*/

static void on_cus_evt(ble_cus_t     * p_cus_service,
                       ble_cus_evt_t * p_evt)
{

    switch(p_evt->evt_type)
    {
        case BLE_CUS_EVT_NOTIFICATION_ENABLED:
            break;

        case BLE_CUS_EVT_NOTIFICATION_DISABLED:
            break;

        case BLE_CUS_EVT_CONNECTED :
            break;

        case BLE_CUS_EVT_DISCONNECTED:
            break;

        default:
              // No implementation needed.
              break;
    }
}
```

プロジェクトをコンパイルし、nRF5x DK にフラッシュします。 nRF Connect アプリを開いてデバイスに接続すると、下のスクリーン ショットに示すように、クライアント特性構成記述子が特性の下に追加されていることがわかります。

サービス、特性および記述子
---
<img src="https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/images/service_char_desc.png" width="500">

これで、nRF52 DK から nRF Connect アプリにいくつかの値を通知する準備ができました。これを行うには、ble_cus_custom_value_update() を定期的に呼び出すアプリケーション タイマーを作成し、BLE_CUS_EVT_NOTIFICATION_ENABLED イベントを取得したときに開始します。まず、main.c の先頭に次の定義を追加します。

```c
/* This code belongs in main.c*/
#define NOTIFICATION_INTERVAL           APP_TIMER_TICKS(1000)
```

これは、アプリケーション タイマーのタイムアウト間隔になります。次に、main.c の定義の下にある APP_TIMER_DEF を呼び出して、m_notification_timer_id という名前のアプリケーション タイマー ID を作成する必要があります。

```c
/* This code belongs in main.c*/
APP_TIMER_DEF(m_notification_timer_id);
```

このマクロ呼び出しの下で、m_custom_value という符号なしの 8 ビット変数を宣言できます。つまり、

```c
/* This code belongs in main.c*/
static uint8_t m_custom_value = 0;
```

これは、nRF Connect アプリに通知するカスタム値を保持するために使用されます。次に、main.c で timers_init() 関数を見つけ、次のスニペットを追加してアプリケーション タイマーを作成します。

```c
/* This code belongs in timers_init() in main.c*/
    // Create timers.
    err_code = app_timer_create(&m_notification_timer_id, APP_TIMER_MODE_REPEATED, notification_timeout_handler);
    APP_ERROR_CHECK(err_code);
```

これでアプリケーション タイマーができました。次のステップは、BLE_CUS_EVT_NOTIFICATION_ENABLED を受信したときにアプリケーション タイマーを開始することです。つまり、main.c の on_cus_evt イベント ハンドラーの BLE_CUS_EVT_NOTIFICATION_ENABLED ケースの下に次のスニペットを追加します。

```c
/* This code belongs in on_cus_evt() in main.c*/
           err_code = app_timer_start(m_notification_timer_id, NOTIFICATION_INTERVAL, NULL);
           APP_ERROR_CHECK(err_code);
```

同様に、BLE_CUS_EVT_NOTIFICATION_DISABLED イベントを取得したときに通知タイマーを停止させたいと考えています。したがって、main.c の on_cus_evt イベント ハンドラーの BLE_CUS_EVT_NOTIFICATION_DISABLED ケースに次のスニペットを追加します。

```c
/* This code belongs in on_cus_evt() in main.c*/
           err_code = app_timer_stop(m_notification_timer_id);
           APP_ERROR_CHECK(err_code);
```

最後に、m_custom_value 変数をインクリメントし、main.c の timers_init() の上にある次の関数を宣言して ble_cus_custom_value_update を呼び出す notification_timeout_handler を作成する必要があります。

```c
/* This code belongs in main.c*/

/**@brief Function for handling the Battery measurement timer timeout.
 *
 * @details This function will be called each time the battery level measurement timer expires.
 *
 * @param[in] p_context  Pointer used for passing some arbitrary information (context) from the
    *                       app_start_timer() call to the timeout handler.
 */
static void notification_timeout_handler(void * p_context)
{
    UNUSED_PARAMETER(p_context);
    ret_code_t err_code;
    
    // Increment the value of m_custom_value before nortifing it.
    m_custom_value++;
    
    err_code = ble_cus_custom_value_update(&m_cus, m_custom_value);
    APP_ERROR_CHECK(err_code);
}
```

プロジェクトをコンパイルし、nRF52 DK にフラッシュします。 nRF Connect アプリを開き、nRF52 DK に接続し、下のスクリーンショットで強調表示されているボタンをクリックして通知を有効にします。

通知を有効にします
---
<img src="https://github.com/NordicPlayground/nRF52-Bluetooth-Course/blob/master/images/enable_notif.png" width="500">

不明な特性プロパティの下に値フィールドが表示され、値が毎秒増加しているのが見えるはずです。これでカスタム サービスが作成され、カスタム値が通知されました。

## タスク 2 - BLE を使用したサーボの制御

このタスクでは、カスタム サービスとその特性を使用して、パルス幅変調を使用してサーボを制御します。 nRF SDK の PWM ライブラリは、PPI および GPIOTE ペリフェラルに加えて、nRF52 の TIMER ペリフェラルの 1 つを使用します。 app_pwm ライブラリは、この Infocenter ページに文書化されています。

### ステップ 1 - サーボを nRF52 DK に接続する

SG90 サーボからの 3 本のワイヤーがあります。

茶色: グラウンド - nRF52 DK の GND とマークされたピンの 1 つに接続する必要があります。

赤: 5V - nRF52 DK の 5V とマークされたピンに接続する必要があります。

オレンジ: PWM 制御信号 - nRF52 DK の未使用の GPIO ピンのいずれかに接続する必要があります (たとえば、P0.04、ピン番号 4)。

### ステップ 2 - 必要なライブラリとドライバーの追加

ライブラリを使用するには、必要な .c および .h ファイルをプロジェクトに追加する必要があります。

app_pwm.h - main.c の先頭にある他の include ステートメントに #include "app_pwm.h" を追加します。

app_pwm.c - プロジェクト エクスプローラーで nRF_Libraries フォルダーを右クリックし、[既存のファイルを追加...] を選択します。 nRF5_SDK_15.0.0_a53641a\components\libraries\pwm フォルダーに移動し、app_pwm.c を追加します。

nrf_drv_ppi.c - プロジェクト エクスプローラーで nRF_Drivers フォルダーを右クリックし、[既存のファイルを追加...] を選択します。 nRF5_SDK_15.0.0_a53641a\integration\nrfx\legacy フォルダーに移動し、nrf_drv_ppi.c を追加します。

nrfx_timer.c - プロジェクト エクスプローラーで nRF_Drivers フォルダーを右クリックし、[既存のファイルを追加...] を選択します。 nRF5_SDK\nRF5_SDK_15.0.0_a53641a\modules\nrfx\drivers\src フォルダーに移動し、nrfx_timer.c を追加します。

nrfx_ppi.c - プロジェクト エクスプローラーで nRF_Drivers フォルダーを右クリックし、[既存のファイルを追加...] を選択します。 nRF5_SDK\nRF5_SDK_15.0.0_a53641a\modules\nrfx\drivers\src フォルダーに移動し、nrfx_ppi.c を追加します。

### 3 - sdk_config.h で追加されたライブラリとドライバーを有効にする

プロジェクトのビルド時にモジュールをコンパイルするには、sdk_config.h ファイルでモジュールを有効にする必要があります。 sdk_config ファイルを開き、以下の定義を検索して、値を 1 に設定します。

sdk_config.h の nRF_Drivers

```
PPI_ENABLED 1

TIMER_ENABLED 1

TIMER1_ENABLED 1
```

sdk_config.h の nRF_Libraries

```
APP_PWM_ENABLED 1
```

### ステップ 3 - main.c への PWM コードの追加

まず、TIMER 1 を使用して PWM 信号を生成する PWM インスタンスを作成する必要があります。これは、BLE_CUS_DEF(m_cus) を追加したのと同じ場所にある main.c に次のパラメーターを含む APP_PWM_INSTANCE マクロを追加することによって行われます。

```c
    //PMW instance
    APP_PWM_INSTANCE(PWM1, 1); // Setup a PWM instance with TIMER 1
```

次に、pwm ライブラリを初期化する pwm_init という関数を作成する必要があります。 main.c の main() のすぐ上に次の関数を追加します。

```c
static void pwm_init()
{
    ret_code_t err_code;
    
    // Configure the PWM library
    app_pwm_config_t pwm1_cfg = {
      .pins            = {4, APP_PWM_NOPIN},                                             // Use pin 4 as the PWM pin and setting the second pin to APP_PWM_NOPIN, i.e. not used
      .pin_polarity    = {APP_PWM_POLARITY_ACTIVE_HIGH, APP_PWM_POLARITY_ACTIVE_LOW},    // Set the polarity of the pin to high, i.e. if the duty cycle is set to 10% the pin will be high 10% of the time and off 90%
      .num_of_channels = 1,                                                             // Set the channel number, i.e. the number of pins to 1
      .period_us       = 20000L                                                         // Set the PWM period to 20msec.
    };

    err_code = app_pwm_init(&PWM1,&pwm1_cfg,NULL);
    APP_ERROR_CHECK(err_code);

    app_pwm_enable(&PWM1);
}
```

最後に、main() で pwm_init() を呼び出し、peer_manager_init() の後に追加する必要があります。

サーボが NRF52 DK に正しく接続され、デューティ サイクルを変更できることを確認するには、main() の for(;;) ループの前に次の行を追加します。

APP_ERROR_CHECK(app_pwm_channel_duty_set(&amp;PWM1,0,7));

これでサーボが90度動きますので、DKに接続する前にサーボアームが外側の位置にあることを確認してください。 pwm_init() と main() 関数の上のコード行を追加すると、次のようになります。

```c
int main(void)
{
    bool erase_bonds;

    // Initialize.
    log_init();
    timers_init();
    buttons_leds_init(&erase_bonds);
    power_management_init();
    ble_stack_init();
    gap_params_init();
    gatt_init();
    services_init();
    advertising_init();
    conn_params_init();
    peer_manager_init();
    pwm_init();

    // Start execution.
    NRF_LOG_INFO("Template example started.");
    application_timers_start();

    advertising_start(erase_bonds);

    // Comment out this line after you have verified that the servo moves to the 90 degree position.
    APP_ERROR_CHECK(app_pwm_channel_duty_set(&PWM1, 0, 7));

    // Enter main loop.
    for (;;)
    {
        idle_state_handle();
    }
}
```

コードをコンパイルして nRF52 DK にフラッシュし、サーボが 90 度の位置に移動することを確認します。サーボが正しく接続されていることを確認したら、その行をコメントアウトします。

### ステップ 4 - サーボ イベントを処理するようにカスタム サービスを変更する

ここで、Bluetooth を介してサーボのデューティ サイクルを受信するようにカスタム サービスを変更します。つまり、デューティ サイクルをカスタム値の特性に書き込み、app_pwm_channel_duty_set() を呼び出して、サーボを選択した位置に移動します。

まず、ble_cus.h の ble_cus_evt_type_t に BLE_CUS_EVT_SERVO イベントを追加します。

```c
/**@brief Custom Service event type. */
typedef enum
{
    BLE_CUS_EVT_NOTIFICATION_ENABLED,                             /**< Custom value notification enabled event. */
    BLE_CUS_EVT_NOTIFICATION_DISABLED,                             /**< Custom value notification disabled event. */
    BLE_CUS_EVT_DISCONNECTED,
    BLE_CUS_EVT_CONNECTED,
    BLE_CUS_EVT_SERVO
} ble_cus_evt_type_t;
```

ble_cus.h で実行したい 2 番目の変更は、duty_cycle 変数を ble_cus_evt_t 構造体に追加して、デューティ サイクル値を main.c の on_cus_evt イベント ハンドラーに渡すことです。つまり、

```c
    /**@brief Custom Service event. */
    typedef struct
    {
        ble_cus_evt_type_t evt_type;                                  /**< Type of event. */
        uint8_t duty_cycle;                                           /**< Servo duty cycle */
    } ble_cus_evt_t;
```

次に行う必要があるのは、BLE で送信されたデューティ サイクルを受信するように ble_cus.c 関数の on_write() を変更することです。

```c
 // Custom Value Characteristic Written to.
    if (p_evt_write->handle == p_cus->custom_value_handles.value_handle)
    {
        nrf_gpio_pin_toggle(LED_4);
        
        // Check if data pointer is not a null-pointer
        if(p_evt_write->data != NULL)
        {
           evt.evt_type = BLE_CUS_EVT_SERVO;
           
           // Set duty cycle to value written to the characteristic
           evt.duty_cycle = p_evt_write->data[0];

           NRF_LOG_INFO("New Duty Cycle received %d\r\n", evt.duty_cycle);

           // Call the application event handler.
           p_cus->evt_handler(p_cus, &evt);
    }
```

変更後、ble_cus.c の on_write() は次のようになります。

```c
    static void on_write(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
    {
        ble_gatts_evt_write_t const * p_evt_write = &p_ble_evt->evt.gatts_evt.params.write;
        ble_cus_evt_t evt;

        // Check if its the Custom Value Characteristic thats been written to.
        if (p_evt_write->handle == p_cus->custom_value_handles.value_handle)
        {
            nrf_gpio_pin_toggle(LED_4);
            
            if(p_evt_write->data != NULL)
            {
                evt.evt_type = BLE_CUS_EVT_SERVO;
                
                evt.duty_cycle = p_evt_write->data[0];

                //Print the calulated duty cycle to the debug terminal
                NRF_LOG_INFO("Duty Cycle: %d \r\n", evt.duty_cycle);

                // Call the application event handler.
                p_cus->evt_handler(p_cus, &evt);
            }
        }

        // Check if the Custom value CCCD is written to and that the value is the appropriate length, i.e 2 bytes.
        if ((p_evt_write->handle == p_cus->custom_value_handles.cccd_handle)
            && (p_evt_write->len == 2)
        )
        {
            // CCCD written, call application event handler
            if (p_cus->evt_handler != NULL)
            {
                
                if (ble_srv_is_notification_enabled(p_evt_write->data))
                {
                    evt.evt_type = BLE_CUS_EVT_NOTIFICATION_ENABLED;
                }
                else
                {
                    evt.evt_type = BLE_CUS_EVT_NOTIFICATION_DISABLED;
                }
                // Call the application event handler.
                p_cus->evt_handler(p_cus, &evt);
            }
        }

    }
```

### ステップ 5 - on_cus_evt() で BLE_CUS_EVT_SERVO イベントを処理する

最後に、main.c の on_cus_evt() のスイッチ ケースに BLE_CUS_EVT_SERVO イベントを追加する必要があります。

```c
    static void on_cus_evt(ble_cus_t     * p_cus_service,
                        ble_cus_evt_t * p_evt)
    {
        ret_code_t err_code;
        
        switch(p_evt->evt_type)
        {
            case BLE_CUS_EVT_NOTIFICATION_ENABLED:
                
                err_code = app_timer_start(m_notification_timer_id, NOTIFICATION_INTERVAL, NULL);
                APP_ERROR_CHECK(err_code);
                break;

            case BLE_CUS_EVT_NOTIFICATION_DISABLED:

                err_code = app_timer_stop(m_notification_timer_id);
                APP_ERROR_CHECK(err_code);
                break;

            case BLE_CUS_EVT_CONNECTED:
                break;

            case BLE_CUS_EVT_DISCONNECTED:
                break;
            // Add this case
            case BLE_CUS_EVT_SERVO:
                break;

            default:
                // No implementation needed.
                break;
        }
    }
```

ここで最後に行う必要があるのは、デューティ サイクルを設定する関数 app_pwm_channel_duty_set() を追加することです。この関数は、p_evt 構造体で渡される Duty_cycle パラメーターを使用します。

```c
static void on_cus_evt(ble_cus_t     * p_cus_service,
                       ble_cus_evt_t * p_evt)
{
    ret_code_t err_code;
    
    switch(p_evt->evt_type)
    {
        case BLE_CUS_EVT_NOTIFICATION_ENABLED:
            
             err_code = app_timer_start(m_notification_timer_id, NOTIFICATION_INTERVAL, NULL);
             APP_ERROR_CHECK(err_code);
             break;

        case BLE_CUS_EVT_NOTIFICATION_DISABLED:

            err_code = app_timer_stop(m_notification_timer_id);
            APP_ERROR_CHECK(err_code);
            break;

        case BLE_CUS_EVT_CONNECTED:
            break;

        case BLE_CUS_EVT_DISCONNECTED:
              break;
        case BLE_CUS_EVT_SERVO:
              err_code = app_pwm_channel_duty_set(&PWM1, 0, p_evt->duty_cycle);
              APP_ERROR_CHECK(err_code);
              break;

        default:
              // No implementation needed.
              break;
    }
}
```

これで、デューティ サイクル値を nRF52 DK に送信し、サーボの角度を変更できるようになります。コードをコンパイルして DK にフラッシュし、nRF Connect アプリを開きます。 nRF52 DK に接続し、特性に 3 ～ 13 の値を書き込みます。注: 16 進数では、3 = 0x03 および 13 = 0xD。

### ステップ 7 - 角度からのデューティ サイクルの計算

BLE を介してデューティ サイクルを送信することは機能しますが、角度を送信し、それに基づいて nRF52832 にデューティ サイクルを計算させることができれば、さらに良いでしょう。

デューティ サイクルを 3% に設定するとサーボは 0 度になり、デューティ サイクルを 13% に設定するとサーボは 180 度になることがわかっています。この情報に基づいて、フォームに線形方程式を設定できます。

```
y = mx + b
```

ここで、m = y2-y1/x2-x1 および b = y(0)、これを参照 [https://www.mathplanet.com/education/algebra-1/formulating-linear-equations/writing-linear-equations-using -the-slope-intercept-form] 理論について。したがって、デューティ サイクルが Y で角度が X である場合、次の式になります。

```
Duty Cycle = m*Angle + b
```

デューティ サイクル (角度=0) = 3 およびデューティ サイクル (角度 = 180) = 13 という事実に基づいて、m と b の値を計算します。

on_write() 関数の evt.duty_cycle = (p_evt_write-&gt;data[0]) を式に置き換えます。コードをコンパイルして DK にフラッシュし、nRF Connect アプリを開きます。 nRF52 DK に接続し、0 から 180 までの値を特性に書き込みます。注: 16 進数では 0 = 0x00、90 = 0x5A、180 = 0xB4。
