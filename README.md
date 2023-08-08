# Session token is invalid
  - Please make sure your __layouts.blade.php__ has in head section:
  ```
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>{{ \Osiset\ShopifyApp\Util::getShopifyConfig('app_name') }}</title>
  ```
  - in body section:
  ```
    @if(\Osiset\ShopifyApp\Util::getShopifyConfig('appbridge_enabled'))
            <script src="https://unpkg.com/@shopify/app-bridge{{ \Osiset\ShopifyApp\Util::getShopifyConfig('appbridge_version') ? '@'.config('shopify-app.appbridge_version') : '' }}"></script>
            <script src="https://unpkg.com/@shopify/app-bridge-utils{{ \Osiset\ShopifyApp\Util::getShopifyConfig('appbridge_version') ? '@'.config('shopify-app.appbridge_version') : '' }}"></script>
            <script
                @if(\Osiset\ShopifyApp\Util::getShopifyConfig('turbo_enabled'))
                    data-turbolinks-eval="false"
                @endif
            >
                var AppBridge = window['app-bridge'];
                var actions = AppBridge.actions;
                var utils = window['app-bridge-utils'];
                var createApp = AppBridge.default;
                var app = createApp({
                    apiKey: "{{ \Osiset\ShopifyApp\Util::getShopifyConfig('api_key', $shopDomain ?? Auth::user()->name ) }}",
                    shopOrigin: "{{ $shopDomain ?? Auth::user()->name }}",
                    host: "{{ \Request::get('host') }}",
                    forceRedirect: true,
                });
            </script>

            @include('shopify-app::partials.token_handler')
            @include('shopify-app::partials.flash_messages')
        @endif
  ```
  - You can get sessionToken with __window.sessionToken__

# Get ShopModel Shopify User

  ```
    public function getShopModelShopifyUser($shop_url) : User
    {
        $shopModel = new User() ?? null;
        $shop__domain = $shop_url;
        $shop__ = User::where('name', $shop__domain)->get()->toArray();
        if( $shop__ ){
            if( isset($shop__[0]) ) {
                $shopId = $shop__[0]['id'];
                $shopModel = User::find($shopId);
            }
        }
        return $shopModel;
    }
  ```
  
# Handle ThemePublshed Job
  - 1
  ```
    public function handle()
    {
        $activeThemeId = $this->data->id;
        $this->shopDomain = ShopDomain::fromNative($this->shopDomain);
        // info( json_encode($this->data->id) );
        // info( $this->shopDomain->toNative() );
        $shop__domain = $this->shopDomain->toNative();
        $shop__ = User::where('name', $shop__domain)->get()->toArray();

        if( $shop__ ){
            if( isset($shop__[0]) ) {
                $shopId = $shop__[0]['id'];
                $shop = User::find($shopId);
                $configuration = Configuration::where('shop_url', $shop__domain)->first();
                $previous_theme_id = $configuration->theme_id;
                $url = redirect()->route("handle-themePublishWebhook-request")->getTargetUrl();
                $curl_url = "$url?shop_url=$shop__domain&activeThemeId=$activeThemeId&previous_theme_id=$previous_theme_id";
                $ch = curl_init();
                curl_setopt($ch, CURLOPT_URL, $curl_url);
                curl_setopt($ch, CURLOPT_RETURNTRANSFER, 0);
                curl_setopt($ch, CURLOPT_TIMEOUT_MS, 2000);
                curl_exec($ch);
                curl_close($ch);

            }
        }
    }
  ```
  - 2
  ```
    public function handleThemePublishWebhookRequest(Request $request)
    {
        try{
            info($request->input("activeThemeId") . " --- " .$request->input("shop_url") . " --- " .$request->input("previous_theme_id"));
            $activeThemeId = $request->input("activeThemeId");
            $shop__ = User::where('name', $request->input("shop_url"))->get()->toArray();
            if( $shop__ ){
                if( isset($shop__[0]) ) {
                    $shopId = $shop__[0]['id'];
                    info("Shop_ID" . $shopId);
                    $shop = User::find($shopId);
                    $configuration = Configuration::where('shop_url', $request->input("shop_url"))->first();
                    $previous_theme_id = $configuration->theme_id;
                    if( asset($previous_theme_id) ):
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "templates", "page.rjee_FAQs", "json");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "sections", "rjee_faqs_section", "liquid");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "sections", "rjee-faqs-section", "liquid");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "rjee_bs5", "css.liquid");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "rjee_bs5", "css");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "Rjee_FAQ_font_awesome_solid", "css.liquid");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "Rjee_FAQ_font_awesome_solid", "css");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "custom_popup_jquery", "js.liquid");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "custom_popup_jquery", "js");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "rjee_bs5_bundle", "js.liquid");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "rjee_bs5_bundle", "js");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "Rjee_FAQ_font_awesome", "js.liquid");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "Rjee_FAQ_font_awesome", "js");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "Rjee_FAQ_font_awesome_solid", "js.liquid");
                        $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "Rjee_FAQ_font_awesome_solid", "js");
                        info("Done deleting Assets");
                        /*
                        $assetDeleted = $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "fa-brands-400", "ttf");
                        $assetDeleted = $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "fa-brands-400", "woff2");
                        $assetDeleted = $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "fa-regular-400", "ttf");
                        $assetDeleted = $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "fa-regular-400", "woff2");
                        $assetDeleted = $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "fa-solid-900", "ttf");
                        $assetDeleted = $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "fa-solid-900", "woff2");
                        $assetDeleted = $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "fa-v4compatibility", "ttf");
                        $assetDeleted = $this->EmptyOutAssetsFromLiquidFile($shop, $previous_theme_id, "assets", "fa-v4compatibility", "woff2");
                        */

                        //if( isset($assetDeleted['body']['container']['message']) ){
                        // $message = $assetDeleted;
                        // info(json_encode($message));
                        // $substring = "was successfully deleted";
                        // if (strpos($message, $substring) != false) {
                        //                            $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "sections", "rjee_faqs_section", "liquid","rjee_faqs_section.liquid");
                        //                            $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "sections", "rjee-faqs-section", "liquid","rjee_faqs_section.liquid");
                        //                            $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "templates", "page.rjee_FAQs", "json","rjee_FAQs.json");
                        //
                        //                            $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "assets", "rjee_bs5", "css.liquid","rjee_bs5.css");
                        //                            $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "assets", "Rjee_FAQ_font_awesome_solid", "css.liquid","Rjee_FAQ_font_awesome_solid.css.liquid");
                        //                            $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "assets", "custom_popup_jquery", "js.liquid","custom_popup_jquery.js.liquid");
                        //                            $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "assets", "rjee_bs5_bundle", "js.liquid","rjee_bs5_bundle.js.liquid");
                        //                            $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "assets", "Rjee_FAQ_font_awesome", "js.liquid","Rjee_FAQ_font_awesome.js.liquid");
                        //                            $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "assets", "Rjee_FAQ_font_awesome_solid", "js.liquid","Rjee_FAQ_font_awesome_solid.js.liquid");

                        $configuration->theme_id = $activeThemeId;
                        if ($configuration->save()) {
                            info("Done Updating Theme ID");
                        }
                        /*
                        $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "assets", "fa-brands-400", "ttf","fa-brands-400.ttf");
                        $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "assets", "fa-regular-400", "ttf","fa-regular-400.ttf");
                        $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "assets", "fa-solid-900", "ttf","fa-solid-900.ttf");
                        $this->CreateAssetsFromLiquidFile($shop, $activeThemeId, "assets", "fa-v4compatibility", "ttf","fa-v4compatibility.ttf");
                        */

                        // } else {
                        //     info("Error! Deleting Asset.");
                        // }
                        // }
                    endif;
                }
            }


        } catch(Exception $e){
            info( "\nMESSAGE :: ". $e->getMessage() ."\nCODE :: ". $e->getCode() ."\nLINE :: ". $e->getLine()  );
        }
        return;
    }

    public function CreateAssetsFromLiquidFile($shop, $activeThemeId, $assetType, $assetFileName, $fileExtension, $fileViewLiquidPathName){
        $file_path = resource_path("views/liquid/$fileViewLiquidPathName");
        $file_contents = file_get_contents($file_path);
        // info( $file_contents );
        $fileURL = "$assetType/$assetFileName.$fileExtension";
        $Data = array(
            'asset' => array(
                'key' => $fileURL,
                'value' => $file_contents
            )
        );
        $shop->api()->rest('PUT', '/admin/themes/' . $activeThemeId . '/assets.json', $Data);
        // info( json_encode($theme_assetCreated) );
    }

    public function EmptyOutAssetsFromLiquidFile($shop, $previousThemeId, $assetType, $assetFileName, $fileExtension){
        $fileURL = "$assetType/$assetFileName.$fileExtension";
        // info($fileURL);
        if($assetType == "templates"){
            $file_path = resource_path("views/liquid/empty.json");
            $file_contents = file_get_contents($file_path);
            $Data = array(
                'asset' => array(
                    'key' => $fileURL,
                    'value' => $file_contents
                )
            );
        }else{
            $Data = array(
                'asset' => array(
                    'key' => $fileURL,
                    'value' => ""
                )
            );
        }
        $theme_assetEmptyOut = $shop->api()->rest('PUT', '/admin/api/themes/' . $previousThemeId . '/assets.json', $Data);
        // info( json_encode($theme_assetEmptyOut) );
    }

  ```
  - 3
  ```
    Route::get('handle-themePublishWebhook-request', [WebhookJobsController::class, 'handleThemePublishWebhookRequest'])->name('handle-themePublishWebhook-request');

  ```

# After Authntication Job
  - https://github.com/ohmybrew/laravel-shopify/wiki/Free-Access-Based-on-Shopify-Plan
  - https://github.com/ohmybrew/laravel-shopify/wiki/After-Authentication-Job
  ```
      'after_authenticate_job' => [

          [
              'job' => \App\Jobs\AfterAuthenticateJob::class, // example: \App\Jobs\AfterAuthenticateJob::class
              'inline' => true // False = execute inline, true = dispatch job for later
          ],

      ],
  ```
  - change \App\Jobs\AfterAuthenticateJob::class handle with
  ```
    public function handle()
    {
        // info($this->shop);
        if ($this->shop->created_at->diffInDays(Carbon::today()) !== 0) {
            // Shop is older than a day, we should've ran this by now, kill it
            return;
        }

        if (!$this->shop->isGrandfathered()) {
            $planName = $this->shop->api()->rest('GET', '/admin/shop.json')['body']['container']['shop'];
            info($planName->plan_name);
            if ($planName->plan_name === 'affiliate' || $planName->plan_name === 'staff_business') {
                $this->shop->shopify_grandfathered = true;
                $this->shop->save();
            }
        }
    }
  ```

# Clear-Log-Laravel-file-with-Artisan

**Here's a reusable artisan command that will save you time and effort :)**

```
  Artisan::command('logs:clear', function() {

      exec('rm -f ' . storage_path('logs/*.log'));

      exec('rm -f ' . base_path('*.log'));

      $this->comment('Logs have been cleared!');

  })->describe('Clear log files');
```
- Drop this in **routes/console.php** then just **run php artisan logs:clear**




<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://cdn.shopify.com/shopifycloud/brochure/assets/brand-assets/shopify-partners-logo-4805ccbfba350cbe454866db370be0d361f65528a1d09c57b1d395e274c37e43.svg" width="400"></a></p>


# Shopify-App-Creation-Guide
-follow these instructions for more details ---> [https://github.com/Kyon147/laravel-shopify](https://github.com/Kyon147/laravel-shopify)
Shopify App Creation Guide


## How To Create Laravel Shopify Apps using Osiset Package
- < https://github.com/osiset/laravel-shopify/wiki/Creating-Webhooks >


- 1: download laragon with php 7.4
- 2: composer create-project --prefer-dist laravel/laravel:^7.0 test_app
- 3: composer require osiset/laravel-shopify:16.* (  for Non-Embend App's )
  ####                                                      OR
- 3: composer require osiset/laravel-shopify:17.* (  for Embend App's )
- 4: php artisan vendor:publish --tag=shopify-config
- 5: https://(your-domain).com/ ( Your app dir URL )
- 6: https://(your-domain).com/authenticate ( in app create section add Rediect's URL )
- 7: edit file ---> routes/web.php and modify the default route
  ```
  Route::get('/', function () { return view('welcome'); })->middleware(['verify.shopify'])->name('home');
  ```
- 8: Modify file --->  resources/views/welcome.blade.php

  ```
  @extends('shopify-app::layouts.default')
  @section('content')
      <!-- You are: (shop domain name) -->
      <p>You are: {{ $shopDomain ?? Auth::user()->name }}</p>
  @endsection
  @section('scripts')
      @parent
      <script>
          actions.TitleBar.create(app, { title: 'Welcome' });
      </script>
  @endsection
  ```

- 9: Edit file ---> app/User.php or app/Models/User.php

  ```
  use Osiset\ShopifyApp\Contracts\ShopModel as IShopModel;
  use Osiset\ShopifyApp\Traits\ShopModel;
  class User extends Authenticatable implements IShopModel
  use ShopModel;
  ```

- 10: For 16.*
  ```
  package use --->  ( auth.shopify  )
  ```
####                           OR
- 10: For 17.*
  ```
  package use --->  ( verify.shopify  )
  ```

- 11:  handle missing domainName exception  goto ---> app\Exceptions\Handler.php  & paste this code their

  ```
  public function render($request, Throwable $exception){
        if( $exception instanceof \Osiset\ShopifyApp\Exceptions\MissingShopDomainException ){
            return response()->view('login', [], 500);
        }
        return parent::render($request, $exception);
  }
  ```

- 12: Create login view ( login.blade.php )

  ```
  <form class="row g-4" action="{{ url('/authenticate') }}" method="GET">
     <div class="input-group mb-3">
         <input name="shop" type="text" class="form-control" placeholder="example.myshopify.com" aria-label="Recipient's username" aria-describedby="button-addon2">
         <button class="btn btn-outline-success" type="submit" id="button-addon2">Install</button>
      </div>
  </form>
  ```

- 13: For Non-embaded app do
  ```
  'appbridge_enabled' => (bool) env('SHOPIFY_APPBRIDGE_ENABLED', false)
  ```
####                          OR
- 13: For Embaded app do
```
'appbridge_enabled' => (bool) env('SHOPIFY_APPBRIDGE_ENABLED', true)
```

- 14: For creating webHooks  {  
  Version 16 used the resource/event format for topic. Version 17 uses the RESOURCE_EVENT format for topic. When upgrading to version 17, make sure to change all of your registered webhooks!

  ```
  php artisan shopify-app:make:webhook [name] [topic]
  / for valid topics --->  
  ```
  refer -->>> https://shopify.dev/api/admin/graphql/reference/events/webhooksubscriptiontopic
  ###              EXAMPLE ::-->   php artisan shopify-app:make:webhook OrdersCreateJob orders/create

- 15: After create webHook we have to config it in {   config/shopify-app.php  } File.
  LIKE :: -->

    ```
        You must create webhookController.php file to bind the webhooks function 
        /*
        |--------------------------------------------------------------------------
        | Shopify Webhooks
        |--------------------------------------------------------------------------
        |
        | This option is for defining webhooks.
        | `topic` is the GraphQL value of the Shopify webhook event.
        | `address` is the endpoint to call.
        |
        | Valid values for `topic` can be found here:
        | https://shopify.dev/api/admin/graphql/reference/events/webhooksubscriptiontopic
        |
        */
                  /* 
                    SHOPIFY_WEBHOOK_1_TOPIC=orders/create
                    SHOPIFY_WEBHOOK_1_ADDRESS="${APP_URL}/webhook/orders-create"

                    SHOPIFY_WEBHOOK_2_TOPIC=themes/publish
                    SHOPIFY_WEBHOOK_2_ADDRESS="${APP_URL}/webhook/themes-publish"

                    SHOPIFY_WEBHOOK_3_TOPIC=app/uninstalled
                    SHOPIFY_WEBHOOK_3_ADDRESS="${APP_URL}/webhook/app-uninstalled"
                   
                 */
      [
         'topic' => env('SHOPIFY_WEBHOOK_0_TOPIC', 'APP_UNINSTALLED'), // APP_UNISTALLED  ===>  "app/uninstalled"
         'address' => env('SHOPIFY_WEBHOOK_0_ADDRESS', 'https://your-domain.com/webhook/app-uninstalled')
      ],
      [
        'topic' => env('SHOPIFY_WEBHOOK_1_TOPIC', 'PRODUCTS_CREATE'), // PRODUCTS_CREATE  ===>  "products/create"
        'address' => env('SHOPIFY_WEBHOOK_1_ADDRESS', 'https://some-app.com/webhook/products-create')
      ],

      ```
        Route::get('/create-webhooks', [WebhookJobsController::class, 'createAllWebhooks'])->name('create_webhooks');
        Route::get('/get-webhooks', [WebhookJobsController::class, 'getAllWebhooks'])->name('get_webhooks');
      ```


###### display_scope
The page or pages on the online store where the script should be included. Valid values:
_Hide display_scope properties_
- **online_store** : Include the script only on the web storefront.
- **order_status**: Include the script only on the order status page.
- **all**: Include the script on both the web storefront and the order status page.

```
        
            
        class Controller extends BaseController
        {
            use AuthorizesRequests, DispatchesJobs, ValidatesRequests;
        
            /**
             * 
             * function to get seleted Theme's ID with Shopify theme API's
             * 
             * @return int $selected_theme_id;
            */
            public function GET__selected_shopify_theme___id($shop , $version){
                $selected_theme_id = null;
                $Get_Theme = $shop->api()->rest("GET" , "/admin/api/". $version ."/themes.json", array() )['body'];
                foreach ($Get_Theme['themes'] as $key => $sel_theme) {
                    if ($sel_theme['role'] == 'main') {
                        $selected_theme_id = $sel_theme;
                    }
                }
                return $selected_theme_id->id;
            }
        
            /**
             * 
             * function to get all Themes with Shopify theme API's
             * 
             * @return array $Get_Themes;
            */
            public function GET__allShopify_themes( $shop , $version ){
        
                $Get_Themes = $shop->api()->rest( "GET" , "/admin/api/". $version ."/themes.json", array() )['body']['themes'];
                return $Get_Themes;
            }
        
            /**
             * 
             * create function to update / create snippet code with Shopify Asset API's
             * 
             * @return array array( 'asset_response' => "" , 'asset_shortcode' => "" );
            */
        
            public function UPDATE_assets__shopifyStore__( $shop , $version, $theme_ID, $snippet_name , $asset_type , $snippet__html_data){
        
                $array = array(
                    "asset" => array(
                        "key" => "layout/theme.liquid"
                    )
                );
                $assets = $shop->api()->rest("GET" , "/admin/api/". $version ."/themes/" . $theme_ID . "/assets.json", $array )['body'];
                
                $theme_liquid_file_Data = urlencode($assets['asset']['value']);
                $snippet_file_name = "{% comment %}Rjee Snippet Start{% endcomment %}\n{% include '$snippet_name' %}\n{% comment %}Rjee Snippet End{% endcomment %}";
                // $head_tag = '</head>';
                // $new_head_tag = $head_tag . $snippet_file_name;
                $head_tag = ('</body>');
                $new_head_tag = urlencode($snippet_file_name . $head_tag);
                $new_theme_liquid_file_Data = str_replace( urlencode($head_tag) , $new_head_tag, $theme_liquid_file_Data);
            
                $OUTPUT_theme_liquid__ = "";
                if (strpos($assets['asset']['value'], $snippet_file_name) === false) {
                    $array = array(
                        "asset" => array(
                            "key" => "layout/theme.liquid",
                            "value" => urldecode($new_theme_liquid_file_Data)
                        )
                    );
                    $assets = $shop->api()->rest("PUT" , "/admin/api/". $version ."/themes/" . $theme_ID . "/assets.json", $array )['body'];
                    // print_r($assets['asset']['value']);
                    $OUTPUT_theme_liquid__ = $assets;
                }
                /////////////////////////////////////////////   Put  snippets --  Creating  countdown-bar.liquid in Store....
                $array = array(
                    'asset' => array(
                        "key" => "". $asset_type ."/". $snippet_name .".liquid",
                        // "value" => '' . file_get_contents(getcwd() . "/". $snippet_name .".txt")
                        "value" => $snippet__html_data
                    )
                );
                $Put_selectedTheme_Assets = $shop->api()->rest("PUT" , "/admin/api/". $version ."/themes/" . $theme_ID . " /assets.json", $array )['body'];
                // return array(	'asset_response' => $Put_selectedTheme_Assets['asset'] , 'asset_shortcode' => urlencode($snippet_file_name)  );
                return json_encode( array(	'asset_response' => $Put_selectedTheme_Assets['asset'] , 'asset_shortcode' => urlencode($snippet_file_name)  ) );
            
            }
            
            /**
             * 
             * Delete function to delete snippet code with Shopify Asset API's
             * 
             * @return array $Delete_selectedTheme_Assets['asset'];
            */
        
            public function DELETE_assets__shopifyStore__( $shop , $version, $theme_ID, $snippet_name , $asset_type ){
                $shop = auth()->user();
                $user_script_tag = CartButtonhiderDetail::where('user_id',  auth()->user()->id)->first();
                // $userArray = User::where('id', $user_script_tag->user_id)->first();
                // dd($userArray->name);
                $array = array(
                    "asset" => array(
                        "key" => "layout/theme.liquid"
                    )
                );
                $assets = $shop->api()->rest("GET", "/admin/api/". $version ."/themes/" . $theme_ID . "/assets.json", $array )['body'];
                $theme_liquid_file_Data = ($assets['asset']['value']);
            
                // $snippet_file_name = "{% include '". $snippet_name ."' %}";
                $snippet_file_name = "{% comment %}Rjee Snippet Start{% endcomment %}\n{% include '$snippet_name' %}\n{% comment %}Rjee Snippet End{% endcomment %}";
                $new_theme_liquid_file_Data = str_replace($snippet_file_name , "", $theme_liquid_file_Data);
            
                $OUTPUT_theme_liquid__ = "";
                if (strpos( ($assets['asset']['value']) , $snippet_file_name) !== false ) {
                    $array = array(
                        "asset" => array(
                            "key" => "layout/theme.liquid",
                            "value" => ($new_theme_liquid_file_Data)
                        )
                    );
                    $assets = $shop->api()->rest("PUT",  "/admin/api/". $version ."/themes/" . $theme_ID . "/assets.json", $array )['body'];
                    // print_r($assets['asset']['value']);
                    $OUTPUT_theme_liquid__ = $assets;
                }
                // print_r($OUTPUT_theme_liquid__);
        
        
                $array = array(
                    'asset' => array(
                        "key" => "". $asset_type ."/". $snippet_name .".liquid",
                        "value" => ""
                    )
                );
                $Delete_selectedTheme_Assets = $shop->api()->rest('PUT', "/admin/api/". $version ."/themes/" . $theme_ID . " /assets.json", $array )['body'];
                return json_encode($Delete_selectedTheme_Assets['asset']);
            }
        
            /**
             * 
             * function to Get script tags from Shopify using Script Tag API's
             * 
             * @return json $script_tag; | []
             * @return String "No Script Tags Found!!!";
            */
        
            public function Get__shopify_ScriptTags($shop , $version){
                
                $shop = auth()->user();
                $version = env('SHOPIFY_API_VERSION');
                $user_script_tag = CartButtonhiderDetail::where('user_id',  auth()->user()->id)->first();
        
                if( !is_null($user_script_tag) ){
                    CartButtonhiderDetail::where('user_id', auth()->user()->id)->update(
                        array(
                            'status' => 0,
                            'shopify_scripttag_id' => NULL
                        )
                    );
                    // /admin/api/2022-04/script_tags.json
                    $script_tag = $shop->api()->rest('GET', "/admin/api/" . $version . "/script_tags.json")['body']['script_tags'];
                    return json_encode( $script_tag );
                }else{
                    // echo "<pre>";
                    return "No Script Tags Found!!!";
                    // echo "</pre>";
                }
            }
        
            /**
             * 
             * function to Get all webHooks using Events API's
             * 
             * @return json $script_tag; | []
             * @return String "No WebHooks Found!!!";
            */
        
            public function Get__installed_webHooks($shop , $version){
                
                $shop = auth()->user();
                $version = env('SHOPIFY_API_VERSION');
                $user_ = CartButtonhiderDetail::where('user_id',  $shop->id)->first();
        
                if( !is_null($user_) ){
                    // /admin/api/2022-04/webhooks.json
                    $webHooks = $shop->api()->rest('GET', "/admin/api/" . $version . "/webhooks.json")['body']['webhooks'];
                    // return json_encode( $script_tag );
                    return ( $webHooks );
                }else{
                    // echo "<pre>";
                    return "No WebHooks Found!!!";
                    // echo "</pre>";
                }
            }
        
            /**
             * 
             * function to Delete all webHooks using Events->webhooks API's
             * 
             * @return json $script_tag; | []
             * @return String "No WebHooks Found!!!";
            */
        
            public function Delete_allInstalled_webHooks($shop , $version){
                
                $shop = auth()->user();
                $version = env('SHOPIFY_API_VERSION');
                $user_ = CartButtonhiderDetail::where('user_id',  $shop->id)->first();
        
                if( !is_null($user_) ){
                    // /admin/api/2022-04/webhooks.json
                    $webHooks = $shop->api()->rest('GET', "/admin/api/" . $version . "/webhooks.json")['body']['webhooks'];
                    // return json_encode( $script_tag );
                    $deleted__webhooksArray = [];
                    foreach ($webHooks as $key => $webHook) {
                        // echo $webHook['id'];
                        $deleted__webhooksArray[] =[ 'webhook_'.$key => $webHook['id'] ];
                        if( !is_null($webHook) ){
                            $shop->api()->rest('DELETE', "/admin/api/" . $version . "/webhooks/". $webHook['id'] .".json");
                        }
                    }
                    return ( $deleted__webhooksArray );
                }
            }
            
            
        
        
            /**
             * 
             * function to reset/ delete script tags from Shopify using Script Tag API's
             * 
             * @return array $Delete_selectedTheme_Assets['asset']; | []
             * @return String "No Script Tags Found!!!";
            */
        
            public function Reset__shopify_ScriptTags($shop , $version){
                
                $shop = auth()->user();
                $version = env('SHOPIFY_API_VERSION');
                $user_script_tag = CartButtonhiderDetail::where('user_id',  auth()->user()->id)->first();
        
                if( !is_null($user_script_tag) ){
                    CartButtonhiderDetail::where('user_id', auth()->user()->id)->update(
                        array(
                            'status' => 0,
                            'shopify_scripttag_id' => NULL
                        )
                    );
                    // /admin/api/2022-04/script_tags.json
                    $script_tag = $shop->api()->rest('GET', "/admin/api/" . $version . "/script_tags.json")['body']['script_tags'];
                    
                    foreach ($script_tag as $key => $s_tags_value) {
                        // echo $s_tags_value['id'];
                        $script_tag = $shop->api()->rest('DELETE', "/admin/api/" . $version . "/script_tags/" . $s_tags_value['id'] . ".json")['body'];
                        // echo "<pre>";
                        // print_r($script_tag);
                        // echo "</pre>";
                    }
                    return "Done Removing Script Tags from shopify Store.";
                }else{
                    // echo "<pre>";
                    return "No Script Tags Created!!!";
                    // echo "</pre>";
                }
            }
        
            public function Create_or_Update_scriptTags($shop , $version , CartButtonhiderDetail $CartButtonhiderDetail , $status , $script_file_url){
        
                $shop = auth()->user();
                // $user_script_tag = CartButtonhiderDetail::where('user_id',  $shop->id)->first();
                $user_script_tag = $CartButtonhiderDetail::where('user_id',  $shop->id)->first();
        
        
                if ( $status == 1 ) {
                    /////////////////////////////////////////////////////////////////////////////////
                    if ($user_script_tag == NULL) {
                        $user_script_info_array = array(
                            "script_tag" => array(
                                "event" => 'onload',
                                // "src" => asset('asset/script_tags/script1.js')
                                "src" => asset($script_file_url)
                            )
                        );
        
                        $script_tag = $shop->api()->rest('POST', "/admin/api/" . $version . "/script_tags.json", $user_script_info_array)['body']['script_tag'];
                        $script_tag_id = $script_tag['id'];
                        $CartButtonhiderDetail::create(
                            array(
                                'user_id' => $shop->id,
                                'status' => $status,
                                'shopify_scripttag_id' => $script_tag_id
                            )
                        );
                        return "Script Tag created!!!";
                    } else {
                        $user_script_info_array = array(
                            "script_tag" => array(
                                "event" => 'onload',
                                // "src" => asset('asset/script_tags/script1.js')
                                "src" => asset($script_file_url)
                            )
                        );
                        $script_tag = $shop->api()->rest('POST', "/admin/api/" . $version . "/script_tags.json", $user_script_info_array)['body']['script_tag'];
                        $script_tag_id = $script_tag['id'];
                        $CartButtonhiderDetail::where('user_id', $shop->id )->update(
                            array(
                                'status' => $status,
                                'shopify_scripttag_id' => $script_tag_id
                            )
                        );
                        return "Script Tag & Data Updated!!!";
                    }
                    /////////////////////////////////////////////////////////////////////////////////
        
                } // end   if( $status == 1 ){
                elseif ($user_script_tag != NULL && $user_script_tag->count()) {
                    // CartButtonhiderDetail::where('user_id', auth()->user()->id)->update(
                        $CartButtonhiderDetail::where('user_id', $shop->id )->update(
                        array(
                            'status' => 0,
                            'shopify_scripttag_id' => NULL
                        )
                    );
                    $script_tag = $shop->api()->rest('DELETE', "/admin/api/" . $version . "/script_tags/" . $user_script_tag->shopify_scripttag_id . ".json")['body'];
                    return "Script Tag Deleted!!!";
                } // end     elseif ($user_script_tag != NULL && $user_script_tag->count()) {
        
        
            }
        
        }
    
```


###### Add it in WebhookJobsController.php controller file

     ```
     
      <?php

        namespace App\Http\Controllers;
        
        use Illuminate\Http\Request;
        
        class WebhookJobsController extends Controller
        {
            //
            public function createAllWebhooks(Request $request)
            {
            try{
            $shop = Auth::user();
            $domain = $shop->getDomain()->toNative();
            $configuration = Configuration::where('shop_url', $domain)->first();
            //return $request->input();
            //die();
            $webhook_array = array(
            "webhook" => array(
            "topic"   => "app/uninstalled",
            "address" =>  env('APP_URL') . "/webhook/app-uninstalled",
            "format"  => "json"
            )
            );
            $webhook_result = $shop->api()->rest('POST', '/admin/api/'. env('SHOPIFY_API_VERSION') .'/webhooks.json', $webhook_array );
            $webhook_array = array(
            "webhook" => array(
            "topic"   => "products/create",
            "address" =>  env('APP_URL') . "/webhook/products-create",
            "format"  => "json"
            )
            );
            $webhook_result = $shop->api()->rest('POST', '/admin/api/'. env('SHOPIFY_API_VERSION') .'/webhooks.json', $webhook_array );
    
                info('Webhooks Created.');
                return redirect()->route('configuration' , array_merge($request->input(), ['request' => $request , 'shop' => $domain]) );
    
            }
            catch(Exception $e){
                return dd( "\nMESSAGE :: ". $e->getMessage() ."\nCODE :: ". $e->getCode() ."\nLINE :: ". $e->getLine()  );
            }
        }
    
        public function checkWebHooks(Request $request)
        {
            try{
                if (isset(auth()->user()->name)) {
                    $shop = Auth::user();
                    $domain = $shop->getDomain()->toNative();
                    $configuration = Configuration::where('shop_url', $domain)->first();
                    if (empty($configuration)) {
                        $this->createAllWebhooks($request);
                        $new_config = new Configuration();
                        $new_config->shop_url = $domain;
                        $new_config->webhook_status = true;
                        if ($new_config->save()) {
                            return redirect()->route('configuration' , array_merge($request->input(), ['request' => $request , 'shop' => $domain]) );
                        }
                    } else {
                        if ($configuration['webhook_status'] == 0) {
                            $this->Create_allWebHooks($request);
                            $edit_config = Configuration::where('shop_url',  $domain)->first();
                            $edit_config->webhook_status = 1;
                            if ($edit_config->update()) {
                                return redirect()->route('configuration' , array_merge($request->input(), ['request' => $request , 'shop' => $domain]));
                            }
                        }
                    }
                }
                return redirect('/configuration');
            }
            catch(Exception $e){
                return dd( "\nMESSAGE :: ". $e->getMessage() ."\nCODE :: ". $e->getCode() ."\nLINE :: ". $e->getLine()  );
            }
        }
    
        public function getAllWebhooks(Request $request)
        {
            try{
                $shop = Auth::user();
                $webhook_result = $shop->api()->rest('GET', '/admin/api/'. env('SHOPIFY_API_VERSION') .'/webhooks.json', array() );
                echo '<pre>';
                print_r( ($webhook_result['body']['container']['webhooks']) );
                echo '</pre>';
            } catch(Exception $e){
                return dd( "\nMESSAGE :: ". $e->getMessage() ."\nCODE :: ". $e->getCode() ."\nLINE :: ". $e->getLine()  );
            }
        }
    }
     ```

### Using REST API's Link  ```  https://github.com/gnikyt/laravel-shopify/wiki/Usage#accessing-api-for-the-current-shop  ```
- // Underlying API package (osiset/basic-shopify-api)
  ```
    $shop = Auth::user();
    $shop->api()->rest(...);
    $shop->api()->graph(...);
    Example:
  ```
  ```
    $shop = Auth::user();
    $request = $shop->api()->rest('GET', '/admin/shop.json');
    // $request = $shop->api()->graph('{ shop { name } }');
    echo $request['body']['shop']['name'];
  ```
- Example with parameters:
  ```
     $shop = Auth::user();
     $request = $shop->api()->rest('GET', '/admin/api/customers/search.json', ['query' => "phone:{$phone}"]);
     echo $request['body']['customers'];
  ```
- Example POST with payload:
   ```
     $shop = Auth::user();
     $request = $shop->api()->rest('POST', '/admin/api/customers/customer.json', ['customer' => "phone:{$phone}"]);
     echo $request['body']['customers'];
   ```
- Accessing Shop's plan charge
  For single/recurring/credit type charges, you can access them via the Charge model's retrieve method for charges which exist in the database for a shop.
  --Example:
  ```
     use Osiset\ShopifyApp\Services\ChargeHelper;

     $shop = Auth::user();
     $chargeHelper = app()->make(ChargeHelper::class);
     $charge = $chargeHelper->chargeForPlan($shop->plan->getId(), $shop);
     $chargeApi = $chargeHelper->useCharge($charge->getReference())->retrieve(); 
  ```  

- 16: Change it in .env file like this
  LIKE :: -->
    ```
      SHOPIFY_WEBHOOK_1_TOPIC=orders/create
      SHOPIFY_WEBHOOK_1_ADDRESS="${APP_URL}/webhook/orders-create"
      SHOPIFY_WEBHOOK_1_TOPIC=themes/publish
      SHOPIFY_WEBHOOK_1_ADDRESS="${APP_URL}/webhook/themes-publish"
    ```

- 17:  ADD  Shopify scopes in the scope in .env file  {  https://shopify.dev/api/usage/access-scopes   }
  LIKE :: -->

- 18: Create an web Route to clear cache  in database for webhooks    {  https://shopify.dev/api/admin-rest/2022-04/resources/webhook   }
  LIKE::-->
    ```
       Route::get('clear-cache', function () {
           Artisan::call('cache:clear');
           Artisan::call('route:clear');
           Artisan::call('config:clear');
           return "Cache is cleared";
       });
       Route::get('fresh-migrate', function () {
           Artisan::call('migrate:fresh');
           return "Migrated is fresh created";
       });
       Route::get('refresh-migrate', function () {
           Artisan::call('migrate:refresh');
           return "Migrated is fresh created";
       });
    ```

- 19:  Add Laravel logs Viewer package to view errors in detailed UI   {  https://github.com/R-jee/laravel-log-viewer  }
  LIKE::-->
    ```
       Install via composer
                 composer require rap2hpoutre/laravel-log-viewer
       Add Service Provider to config/app.php in providers section
                 Rap2hpoutre\LaravelLogViewer\LaravelLogViewerServiceProvider::class,
       Add a route in your web routes file:
                 Route::get('logs', [\Rap2hpoutre\LaravelLogViewer\LogViewerController::class, 'index']);
    ```

- 20:    Log::info($input);

- 21:    php artisan make:model --migration --controller webhookJobs

- 22:  Create Charging Plans
  ###     ---> create seeds in database section PlanSeeder.php
  ```
       <?php
          use Illuminate\Database\Seeder;
          use Osiset\ShopifyApp\Storage\Models\Plan;

          class PlanSeeder extends Seeder
          {
              /**
               * Seed the application's database.
               *
               * @return void
               */
              public function run()
              {
                  // $this->call(UsersTableSeeder::class);

                  /*
                   # Create a recurring "Demo" plan for $5.00, with 7 trial days, which will be presented on install to the shop and have the ability to issue usage charges to a maximum of $10.00
                      INSERT INTO plans (
                          `type`,
                          `name`,
                          `price`,
                          `interval`,
                          `capped_amount`,
                          `terms`,
                          `trial_days`,
                          `test`,
                          `on_install`,
                          `created_at`,
                          `updated_at`) VALUES
                      ('RECURRING/ONETIME','Test Plan',5.00,'EVERY_30_DAYS',10.00,'Test terms',7,FALSE,1,NULL,NULL);
                  */


                  $Plan = new Plan();
                  $Plan->type = "RECURRING";
                  $Plan->name = "Basic Plan";
                  $Plan->price = 4.99;
                  $Plan->interval = "EVERY_30_DAYS";
                  $Plan->capped_amount = 10.00;
                  $Plan->terms = "Basic Plan ~ amount 4.99";
                  $Plan->trial_days = 7;
                  $Plan->test = 1;
                  $Plan->on_install = 1;
                  $Plan->save();

                  $Plan = new Plan();
                  $Plan->type = "RECURRING";
                  $Plan->name = "Premiere Plan ~ amount 9.99";
                  $Plan->price = 9.99;
                  $Plan->interval = "EVERY_30_DAYS";
                  $Plan->capped_amount = 10.00;
                  $Plan->terms = "Premiere Plan ~ amount 9.99";
                  $Plan->trial_days = 14;
                  $Plan->test = 1;
                  $Plan->on_install = 1;
                  $Plan->save();

              }
          }
  ```

- 23 :  php artisan make:seeder PlanSeeder  ------->    php artisan db:seed  ,  php artisan db:seed --class=PlanSeeder

- 24 :  php artisan make:middleware CustomBillable

- 25:
  ```
    <?php
      namespace App\Http\Middleware;
      use Closure;
      use Illuminate\Support\Facades\Config;
      class CustomBillable
      {
          /**
           * Handle an incoming request.
           *
           * @param  \Illuminate\Http\Request  $request
           * @param  \Closure  $next
           * @return mixed
           */
          public function handle($request, Closure $next)
          {
              info(json_encode($request));
              if( Config::get('shopify-app.billing_enabled') === true  ){
                  $shop = auth()->user();

                  if(!$shop->isFreemium() && !$shop->isGrandfathered() && $shop->plan == null ){
                  // if(!$shop->shopify_freemium && !$shop->shopify_grandfathered && $shop->plan == null ){
                      return redirect()->route('billing.plans');
                  }
              }
              return $next($request);
          }
      }
  ```

  ### ADD  middleware in  Kernel.php file at  last
  ```
    'custom.billable' => \App\Http\Middleware\CustomBillable::class,
  ```

- 26:
  ```
    <div class="bottom">
         <a href="{{ route('billing', ['plan' => $plans[0]->id ]) }}">Buy Now</a>
    </div>
    <div class="bottom">
         <a href="{{ route('billing', ['plan' => $plans[1]->id ]) }}">Buy Now</a>
    </div>

    <div class="bottom">
        <a href="{{ route('free.plan') }}">Get Access</a>
    </div>
  ```

- 27 :
  ```
    Route::get('/free-plan', function(){
        User::where('id' , auth()->user()->id )->update(
            [
                'plan_id' => NULL,
                'shopify_freemium' => 1
            ]
        );
        return redirect()->route('home');
    })->name('free.plan');  
  ```

- 28:  App proxy in app
  ```
    Route::get('checkSetupStatus', 'CartButtonhiderDetailController@checkSetupStatus')->name('check.SetupStatus')->middleware(['auth.proxy']);
        public function checkSetupStatus(){
            $shop = User::where('name', request('shop'))->first();
            if($shop->status){
                return response()->json(['status' => true]);
            }else{
                return response()->json(['status' => false]);
            }
        }
  ```

### After Authentication Job -->
- php artisan make:job AfterAuthenticateJob

### If you want to update the password of your user using the shopify API please try to use the below code:-
```
$updatePassword = array(
          "customer"=>array(
              'id'=>$userId,
              'password'=> $updatedPassword,
              'password_confirmation'=>$confirm_updatedPassword,
              'send_reset_password_email': true    

          )
      );
$updateCustomer = $shopify("PUT /admin/customers/$userId.json" , $updatePassword);
```
```
  PUT /admin/customers/#{id}.json
  {
    "customer": {
      "id": 207119551,
      "password": "newpass",
      "password_confirmation": "newpass",
      "send_email_welcome": false
    }
  }
```
### JS Shopify Login

  ```
    var url = decodeURI(window.location.toString());     // This part is not correct
    var arr = url.split('?');
    if (arr.length > 1 && arr[1] !== '') {
        // console.log('params found');
        var firstSplit = url.split('?')[1];
        var email_______ = ( firstSplit.split('&')[0] ).split('=')[1];
        var pass________ = ( firstSplit.split('&')[1] ).split('=')[1];
        var redirect_uri________ = ( firstSplit.split('&')[3] ).split('=')[1];
        /*
        * Try to login, check the login credentials, and then redirect if required.
        */
        login(decodeURIComponent(email_______), decodeURIComponent(pass________), decodeURIComponent(redirect_uri________)).done(function (html) {
            // $('body').html(html);
            if (html.indexOf('Invalid login credentials') !== -1) {
            // invalid password - show a message to the user
            } else {
                // All good! Let's redirect if required
                // window.location.href = window.location.origin;
                var checkoutUrl = getCheckoutUrl();
                if (checkoutUrl) {
                    // window.location = checkoutUrl;
                } else {
                    // don't need to redirect, do what you like!
                    $('body').html(html + `<script src="https://code.jquery.com/jquery-3.6.1.min.js" ></script><script>
                        $(".shopify-challenge__button").on('click', function(e){
                            if( $("#recaptcha-anchor").hasClass("recaptcha-checkbox-checked")  ) {
                                console.log("");
                                window.close();

                            }
                    });</script>`);

                }
            }
        });
    }
    
    /*
    * Log the customer in with their email and password.
    */
    function login(email, password, redirect_uri) {
        var data = {
            'customer[email]': email,
            'customer[password]': password,
            'form_type': 'customer_login',
            'utf8': '✓',
            "return_to": redirect_uri

        };

        var promise = $.ajax({
            url: '/account/login',
            method: 'post',
            data: data,
            dataType: 'html',
            async: true
        });

        return promise;
    }

    /*
    * Get the `checkout_url` from the URL query parameter, if it exists.
    * (It only ever exists on the /account/login page)
    */
    function getCheckoutUrl() {
        function getParameterByName(name) {
        name = name.replace(/[\[]/, "\\[").replace(/[\]]/, "\\]");
        var regex = new RegExp("[\\?&]" + name + "=([^&#]*)"),
            results = regex.exec(location.search);
        return results === null ? "" : decodeURIComponent(results[1].replace(/\+/g, " "));
        }
        return getParameterByName('checkout_url');
    }

  ```


## Enable Billing Process

- vendor/osiset/laravel-shopify/src/Http/Middleware/VerifyShopify.php:96 comment out this line  **throw new SignatureVerificationException('Unable to verify signature.');**

- add custome billable middleware in Middleware folder  --->  ( `CustomBillable.php` )
```
namespace App\Http\Middleware;

       use Closure;
       use Illuminate\Support\Facades\Config;

       class CustomBillable
       {
           /**
            * Handle an incoming request.
            *
            * @param  \Illuminate\Http\Request  $request
            * @param  \Closure  $next
            * @return mixed
            */
           public function handle($request, Closure $next)
           {
               info(json_encode($request));
               if( Config::get('shopify-app.billing_enabled') === true  ){
                   $shop = auth()->user();

                   if(!$shop->isFreemium() && !$shop->isGrandfathered() && $shop->plan == null ){
                   // if(!$shop->shopify_freemium && !$shop->shopify_grandfathered && $shop->plan == null ){
                       return redirect()->route('billing.plans');
                   }
               }
               return $next($request);
           }
       }


    ```

-add custome billable middleware in Kernal   --->  ( `app\Http\kernal.php` )
```
    'custom.billable' => \App\Http\Middleware\CustomBillable::class,
```




-add custome billable view   --->  ( `billing.blade.php` )
```
    //  write your code here


      <div class="container">
          <h1>Choose Your Perfect Plan</h1>
          <div class="block" id="pricing_basic">
              <div class="block_container">
                  <div class="top">
                      <div class="title">Free Plan</div>
                  </div>

                  <div class="middle">
                      <div class="price_amount"><span class="pound_sign">$</span>0.00</div>
                      <div class="price_sub">per month</div>
                      <div class="features">
                          <ul>
                              <li class="feature1">Up to 2 FTP Accounts</li>
                              <li class="feature2">Up to 10 Databases</li>
                              <li class="feature3">Unlimited Data Transfer</li>
                              <li class="feature4">2GB Online Storage</li>
                          </ul>
                      </div>
                  </div>

                  <div class="bottom">
                      <a href="{{ route('free.plan') }}">Get Access</a>
                  </div>
              </div>
          </div>


          <div class="block" id="pricing_advanced">
              <div class="block_container">
                  <div class="top">
                      <div class="title">{{ $plans[0]['name'] }}</div>
                  </div>

                  <div class="middle">
                      <div class="price_amount"><span class="pound_sign">$</span>{{ $plans[0]['price'] }}</div>
                      <div class="price_sub">per month</div>
                      <div class="features">
                          <ul>
                              <li class="feature1">Up to 10 FTP Accounts</li>
                              <li class="feature2">Up to 100 Databases</li>
                              <li class="feature3">Unlimited Data Transfer</li>
                              <li class="feature4">10GB Online Storage</li>
                          </ul>
                      </div>
                  </div>

                  <div class="bottom">
                      <a href="{{ route('billing', ['plan' => $plans[0]->id ]) }}">Buy Now</a>
                  </div>
              </div>
          </div>



          <div class="block" id="pricing_professional">
              <div class="block_container">
                  <div class="top">
                      <div class="title">{{ $plans[1]['name'] }}</div>
                  </div>

                  <div class="middle">
                      <div class="price_amount"><span class="pound_sign">$</span>{{ $plans[1]['price'] }}</div>
                      <div class="price_sub">per month</div>
                      <div class="features">
                          <ul>
                              <li class="feature1">Up to 200 FTP Accounts</li>
                              <li class="feature2">Unlimited Databases</li>
                              <li class="feature3">Unlimited Data Transfer</li>
                              <li class="feature4">50GB Online Storage</li>
                          </ul>
                      </div>
                  </div>

                  <div class="bottom">
                      <a href="{{ route('billing', ['plan' => $plans[1]->id ]) }}">Buy Now</a>
                  </div>
              </div>
          </div>



      </div>

      <style>
          body {
              font: 16px Verdana, Tahoma, Arial, sans-serif;
              background: url('http://cdn.morguefile.com/imageData/public/files/k/kevinrosseel/preview/fldr_2008_11_28/file0001082180876.jpg');
              background-size: cover;
              color: #fff;
          }

          @media all and (max-width: 800px) {
              body {
                  font-size: 15px !important;
              }
          }

          .container {
              width: 85%;
              max-width: 1200px;
              margin: 0 auto;
          }

          .block {
              width: 28%;
              margin: 0 2.5% 0 0;
              float: left;
          }

          .block:last-child {
              margin-right: 0 !important;
          }

          .block_container>div {
              padding: 8% 5% 2%;
          }

          .block#pricing_basic>div>div {
              background-color: #ef8006;
          }

          .block#pricing_advanced>div>div {
              background-color: #77bf22;
          }

          .block#pricing_professional>div>div {
              background-color: #1a8aca;
          }

          .block_container {}

          /* --------------- TOP --------------- */
          .top {}

          .title {
              font-weight: bold;
              font-size: 2vw;
              text-align: center;
              line-height: 4vw;
          }

          /* -------------- MIDDLE ------------- */
          .middle {
              height: 280px;
          }

          @media all and (min-width: 501px) and (max-width: 640px) {
              .middle {
                  height: 300px;
              }
          }

          @media all and (max-width: 500px) {
              body {
                  font-size: 17.5px !important;
              }

              .block {
                  width: 100%;
                  margin: 6vh 0;
              }

              .title {
                  font-size:6vw;
                  line-height: 10vw;
              }

              .price_amount {
                  font-size: 15vw !important;
                  line-height: 20vw !important;
                  height: 18vw !important;
              }

              .pound_sign {
                  font-size: 8vw !important;
                  line-height: 16vw !important;
              }

              .price_sub {
                  font-size: 6vw !important;
              }

              .middle {
                  height: auto;
              }

              .bottom {
                  margin-top: 12px !important;
              }
          }

          .price_amount {
              font-weight: bold;
              font-size: 6vw;
              line-height: 6vw;
              letter-spacing: -.4vw;
              text-align: center;
              height: 7vw;
          }

          .pound_sign {
              font-size: 3vw;
              line-height: 4vw;
              vertical-align: top;
              margin-right: 1vw;
          }

          .price_sub {
              font-weight: bold;
              font-size: 2vw;
              text-align: center;
          }

          .features {}

          .features ul {
              padding-left: 1.5em;
          }

          .features ul li {}

          .feature1 {}

          .feature2 {}

          .feature3 {}

          .feature4 {}

          /* -------------- BOTTOM ------------- */
          .bottom {
              margin-top: 30px;
              padding: 7% 10% !important;
              text-align: center;
          }

          .bottom a {
              color: #fff;
              text-decoration: none;
          }
      </style>
       
       
    ```

-add custome billable middleware in Middleware folder  --->  ( `CustomBillable.php` )

     ```
        Route::middleware(['auth.shopify'])->group(function(){
           Route::get('/', function () {
               $data = CartButtonhiderDetail::where('user_id', auth()->user()->id )->first();
               return view('welcome', compact('data'));
           // })->name('home')->middleware(['custom.billable']);
           })->name('home')->middleware(['custom.billable']);

           Route::get('/login', function(){
               return view('login');
           })->name('login');

           // to show all billing plans
           Route::get('/plans', function(){
               $plans = Plan::all();
               return view('billing')->with(compact( 'plans' ));
           })->name('billing.plans');

           // to get free plan 
           Route::get('/free-plan', [CartButtonhiderDetailController::class , 'freePlan'])->name('free.plan');
       });    

   ```
 
 -add plans seeder in DatabaseSeeder.php 
 ```
     <?php

     namespace Database\Seeders;

     use Illuminate\Database\Seeder;
     use Osiset\ShopifyApp\Storage\Models\Plan;

     class DatabaseSeeder extends Seeder
     {
       /**
        * Seed the application's database.
        *
        * @return void
        */
       public function run()
       {
           // \App\Models\User::factory(10)->create();

           $Plan = new Plan();
           $Plan->type = "RECURRING";
           $Plan->name = "Basic Plan";
           $Plan->price = 4.99;
           $Plan->interval = "EVERY_30_DAYS";
           $Plan->capped_amount = 10.00;
           $Plan->terms = "Basic Plan ~ amount 4.99";
           $Plan->trial_days = 7;
           $Plan->test = 1;
           $Plan->on_install = 1;
           $Plan->save();

           $Plan = new Plan();
           $Plan->type = "RECURRING";
           $Plan->name = "Premiere Plan ~ amount 9.99";
           $Plan->price = 9.99;
           $Plan->interval = "EVERY_30_DAYS";
           $Plan->capped_amount = 10.00;
           $Plan->terms = "Premiere Plan ~ amount 9.99";
           $Plan->trial_days = 14;
           $Plan->test = 1;
           $Plan->on_install = 1;
           $Plan->save();
        }
    }

```
 php artisan db:seed --class=PlanSeeder

 Route::get('/p1/plans-seeders', function () {
     Artisan::call('db:seed --class=PlanSeeder');
     return "Seeder Creation has been done.";
 });
```
-- add these in .env file
SHOPIFY_APPBRIDGE_ENABLED=true
SHOPIFY_BILLING_ENABLED=1

### InvalidArgumentException
--Can only instantiate this object with a string.
--https://linkify.console.autooutletllc.com/billing/1

         ```
          // use it in plans blade view
            <a href="{{ route('billing', ['plan' => $plans[1]->id, 'shop' => $user->name ] ) }}?shop={{ $user->name }}">Buy Now</a>

         ```

### Cancel Change & remove previous change
   ```
       $shop = Auth::user();
            $shop->api()->rest('DELETE', "/admin/api/". env('SHOPIFY_API_VERSION') ."/recurring_application_charges/". $cancelRecurring_application_charges->id .".json", []);
            DB::table('charges')->where([ 'user_id' => auth()->user()->id, 'status' => "ACTIVE"] )->update(
                array(
                    'cancelled_on' => date('Y-m-d 00:00:00'),
                    'expires_on' => date('Y-m-d 00:00:00'),
                    'status' => "CANCELLED"
                )
            );
   ```

### Free Plan Active function
   ```
      public function freePlan( Request $request, IShopCommand $shopCommand , IShopQuery $shopQuery, CancelCurrentPlan $cancelCurrentPlanAction ){
        try{
            $shop = Auth::user();
            $cancelRecurring_application_charges = DB::table('charges')->where('user_id' , $shop->id)->where('status' , "ACTIVE")->where('plan_id', $shop->plan_id)->first();
            if( $cancelRecurring_application_charges ){
                $charge_canceled = $shop->api()->rest('DELETE', "/admin/api/". env('SHOPIFY_API_VERSION') ."/recurring_application_charges/". $cancelRecurring_application_charges->charge_id .".json", array());
                $get_charge_data = $shop->api()->rest('PUT', "/admin/api/". env('SHOPIFY_API_VERSION') ."/recurring_application_charges/". $cancelRecurring_application_charges->charge_id ."/customize.json", array(
                    "recurring_application_charge" => [
                        'status' => "cancelled"
                    ]
                ));
                DB::table('charges')->where([ 'user_id' => auth()->user()->id, 'status' => "ACTIVE"] )->update(
                    array(
                        'cancelled_on' => date('Y-m-d 00:00:00'),
                        'expires_on' => date('Y-m-d 00:00:00'),
                        'trial_days' => 0,
                        'status' => "CANCELLED"
                    )
                );
                User::where('id' , auth()->user()->id )->update([
                    'plan_id' => NULL,
                    'shopify_freemium' => 1
                ]);
            }else{
                User::where('id' , auth()->user()->id )->update([
                    'plan_id' => NULL,
                    'shopify_freemium' => 1
                ]);
            }
            return redirect()->route('home', array_merge($request->input(), ['request' => $request, 'shop' => $shop->name]) );
            // return $this->index($request);
        }
        catch(Exception $e){
            return dd( "\nMESSAGE :: ". $e->getMessage() ."\nCODE :: ". $e->getCode() ."\nLINE :: ". $e->getLine()  );
        }
    }
   ```

### Get Charge Status

 ```
  use Osiset\ShopifyApp\Services\ChargeHelper;
  use Osiset\ShopifyApp\Objects\Values\ChargeReference;
  use App\User;

  // ...

  // Setup the "Charge Helper"
  $chs = resolve(ChargeHelper::class);
  $shops = User::all();

  // Loop all shops
  foreach ($shops as $shop) {
    $plan = $shop->plan;
    if ($plan === null) {
       // No plan, skip
       continue;
    }

    // Get the charge entry from database, set it to the charge helper to use for API calls
    $charge = $chs->chargeForPlan($plan->getId(), $shop);
    $chs->useCharge($charge->getReference());

    // Get the charge data from Shopify API
    $chargeData = $chs->retrieve($shop);

    // Now you can access `$chargeData['status']` to check its status and control your flow
  }
 ```

### Bypass Charge Plan for payment
- go to --> vendor\osiset\laravel-shopify\src\Http\Middleware\VerifyShopify.php file & comment out this line on line number #96
   ```
        // Verify the HMAC (if available)
        $hmacResult = $this->verifyHmac($request);
        if ($hmacResult === false) {
            // Invalid HMAC
            // throw new SignatureVerificationException('Unable to verify signature.');
        }
   ```


## {{  --CREATE SHOPIFY APP EXTENSION USING CLI ---   }}
https://magecomp.com/blog/create-theme-app-extension-shopify/

- 1:  install gem  --> https://rubyinstaller.org/downloads/
- 2: install  git  -->  https://git-scm.com/downloads/
- 3: install Xaamp ( php -v  )
- 4: install node  ( npm -v  )
- 5: install composer ( composer -v )

- 6: gem install shopify-cli  ( for test -->  shopify version )
- 7: shopify login ( for text is it working fine )    |&|    shopify login --store=https://rjee-test-app.myshopify.com/admin

```
   shopify login

   shopify extension create    OR     shopify extension create --type=THEME_APP_EXTENSION --getting-started

   cd theme-app-extension
   shopify extension register

   You can only create one Theme App Extension per app, which can’t be undone.
   ┃ &nbsp; ? Would you like to register this extension? (y/n) &nbsp; (You chose: yes)

   shopify extension push
```

### Create/Update scriptTags
```
    public function showHideButton()
    {
        // echo env("SHOPIFY_API_VERSION");
        // dd(auth()->user());
        $shop = auth()->user();
        $version = env("SHOPIFY_API_VERSION");
        // $user_script_tag = CartButtonhiderDetail::where('user_id',  auth()->user()->id)->first();
        $CartButtonhiderDetail = new CartButtonhiderDetail;

        $script_file_Nmae = "script2.js";
        $script_file_url = "asset/script_tags/$script_file_Nmae";
        $status = request('status');
        echo $this->Create_or_Update_scriptTags($shop, $version, $CartButtonhiderDetail, $status, $script_file_url);
    } // end     function showHideButton()

```
### Remove/Reset all scriptTags
```
    public function resetScriptTags()
    {
        $shop = auth()->user();
        echo $this->Reset__shopify_ScriptTags($shop, env("SHOPIFY_API_VERSION"));
    }
    // to get scriptTags json data
    public function getScriptTags()
    {
        $shop = auth()->user();
        if( $shop ){
            echo $this->Get__shopify_ScriptTags($shop, env("SHOPIFY_API_VERSION"));
        }else{
            return redirect()->route('login');
        }
    }
```

### Create snippet
```
    public function create_snippet()
    {
        $snippetName = "hide_ATC_btn";
        $asset_type_ = "snippets";
        $shop = auth()->user();
        $version = env("SHOPIFY_API_VERSION");
        $user_settings = CartButtonhiderDetail::where('user_id',  auth()->user()->id)->first();
        $active_theme = "";
        $active_theme_id = "";
        // $themes = $this->GET__allShopify_themes();
        $active_theme_id = $this->GET__selected_shopify_theme___id($shop, $version);
        if ($active_theme_id != "") :
            $rep_asset_create = $this->UPDATE_assets__shopifyStore__($shop, $version, $active_theme_id, $snippetName, $asset_type_,  file_get_contents(getcwd() . "/asset/snippets/s1.txt"));
            echo $rep_asset_create;
        endif; // end    if($active_theme_id != ""):
    }
```

### Delete snippet
 ```
    public function delete_snippet()
    {
        // echo json_encode($this->shop() );
        $snippetName = "hide_ATC_btn";
        $asset_type_ = "snippets";
        $shop = auth()->user();
        $version = env("SHOPIFY_API_VERSION");
        $user_settings = CartButtonhiderDetail::where('user_id',  auth()->user()->id)->first();
        $active_theme = "";
        $active_theme_id = "";

        // if( $user_settings != NULL ){

        // $themes = $this->GET__allShopify_themes();
        $active_theme_id = $this->GET__selected_shopify_theme___id($shop, $version);
        if ($active_theme_id != "") :
            // $snippet = $shop->api()->rest('PUT', "/admin/api/". env('SHOPIFY_API_VERSION') ."/themes/". $active_theme_id ."/assets.json" , $create_snippet_assetData)['body']['asset'];
            $rep_asset_delete = $this->DELETE_assets__shopifyStore__($shop, $version, $active_theme_id, $snippetName, $asset_type_);
            echo $rep_asset_delete;
        endif; // end    if($active_theme_id != ""):
        // }

    }
```

###  Get All webhooks
  ```
    public function getWebHooks__all(){
        $shop = auth()->user();
        $version = env("SHOPIFY_API_VERSION");
        if ($shop !== NULL) {
            dd( $this->Get__installed_webHooks($shop, $version) );
        } else {
            return view('login');
            // dd("User do not have access for these files...");
        }
    }
```

### Delete All webhooks
```
    public function deleteWebHooks__all(){
        $shop = auth()->user();
        $version = env("SHOPIFY_API_VERSION");
        if ($shop !== NULL) {
            dd( $this->Delete_allInstalled_webHooks($shop, $version) );
        } else {
            return view('login');
            // dd("User do not have access for these files...");
        }
    }
```
#### Check status
```
    public function checkSetupStatus(){
        $shop = User::where('name', request('shop'))->first();
        if($shop->status){
            return response()->json(['status' => true]);
        }else{
            return response()->json(['status' => false]);
        }
    }
```


### function to get seleted Theme's ID with Shopify theme API's
```
    public function GET__selected_shopify_theme___id($shop , $version){
        $selected_theme_id = null;
        $Get_Theme = $shop->api()->rest("GET" , "/admin/api/". $version ."/themes.json", array() )['body'];
        foreach ($Get_Theme['themes'] as $key => $sel_theme) {
            if ($sel_theme['role'] == 'main') {
                $selected_theme_id = $sel_theme;
            }
        }
        return $selected_theme_id->id;
    }
```
### function to get all Themes with Shopify theme API's
```
    public function GET__allShopify_themes( $shop , $version ){

        $Get_Themes = $shop->api()->rest( "GET" , "/admin/api/". $version ."/themes.json", array() )['body']['themes'];
        return $Get_Themes;
    }
```

### create function to update / create snippet code with Shopify Asset API's
```
public function UPDATE_assets__shopifyStore__( $shop , $version, $theme_ID, $snippet_name , $asset_type , $snippet__html_data){

        $array = array(
            "asset" => array(
                "key" => "layout/theme.liquid"
            )
        );
        $assets = $shop->api()->rest("GET" , "/admin/api/". $version ."/themes/" . $theme_ID . "/assets.json", $array )['body'];
        
        $theme_liquid_file_Data = urlencode($assets['asset']['value']);
        $snippet_file_name = "{% comment %}Rjee Snippet Start{% endcomment %}\n{% include '$snippet_name' %}\n{% comment %}Rjee Snippet End{% endcomment %}";
        // $head_tag = '</head>';
        // $new_head_tag = $head_tag . $snippet_file_name;
        $head_tag = ('</body>');
        $new_head_tag = urlencode($snippet_file_name . $head_tag);
        $new_theme_liquid_file_Data = str_replace( urlencode($head_tag) , $new_head_tag, $theme_liquid_file_Data);
    
        $OUTPUT_theme_liquid__ = "";
        if (strpos($assets['asset']['value'], $snippet_file_name) === false) {
            $array = array(
                "asset" => array(
                    "key" => "layout/theme.liquid",
                    "value" => urldecode($new_theme_liquid_file_Data)
                )
            );
            $assets = $shop->api()->rest("PUT" , "/admin/api/". $version ."/themes/" . $theme_ID . "/assets.json", $array )['body'];
            // print_r($assets['asset']['value']);
            $OUTPUT_theme_liquid__ = $assets;
        }
        /////////////////////////////////////////////   Put  snippets --  Creating  countdown-bar.liquid in Store....
        $array = array(
            'asset' => array(
                "key" => "". $asset_type ."/". $snippet_name .".liquid",
                // "value" => '' . file_get_contents(getcwd() . "/". $snippet_name .".txt")
                "value" => $snippet__html_data
            )
        );
        $Put_selectedTheme_Assets = $shop->api()->rest("PUT" , "/admin/api/". $version ."/themes/" . $theme_ID . " /assets.json", $array )['body'];
        // return array(	'asset_response' => $Put_selectedTheme_Assets['asset'] , 'asset_shortcode' => urlencode($snippet_file_name)  );
        return json_encode( array(	'asset_response' => $Put_selectedTheme_Assets['asset'] , 'asset_shortcode' => urlencode($snippet_file_name)  ) );
    
    }
   
```
### Delete function to delete snippet code with Shopify Asset API's
```
     public function DELETE_assets__shopifyStore__( $shop , $version, $theme_ID, $snippet_name , $asset_type ){
        $shop = auth()->user();
        $user_script_tag = CartButtonhiderDetail::where('user_id',  auth()->user()->id)->first();
        // $userArray = User::where('id', $user_script_tag->user_id)->first();
        // dd($userArray->name);
        $array = array(
            "asset" => array(
                "key" => "layout/theme.liquid"
            )
        );
        $assets = $shop->api()->rest("GET", "/admin/api/". $version ."/themes/" . $theme_ID . "/assets.json", $array )['body'];
        $theme_liquid_file_Data = ($assets['asset']['value']);
    
        // $snippet_file_name = "{% include '". $snippet_name ."' %}";
        $snippet_file_name = "{% comment %}Rjee Snippet Start{% endcomment %}\n{% include '$snippet_name' %}\n{% comment %}Rjee Snippet End{% endcomment %}";
        $new_theme_liquid_file_Data = str_replace($snippet_file_name , "", $theme_liquid_file_Data);
    
        $OUTPUT_theme_liquid__ = "";
        if (strpos( ($assets['asset']['value']) , $snippet_file_name) !== false ) {
            $array = array(
                "asset" => array(
                    "key" => "layout/theme.liquid",
                    "value" => ($new_theme_liquid_file_Data)
                )
            );
            $assets = $shop->api()->rest("PUT",  "/admin/api/". $version ."/themes/" . $theme_ID . "/assets.json", $array )['body'];
            // print_r($assets['asset']['value']);
            $OUTPUT_theme_liquid__ = $assets;
        }
        // print_r($OUTPUT_theme_liquid__);


        $array = array(
            'asset' => array(
                "key" => "". $asset_type ."/". $snippet_name .".liquid",
                "value" => ""
            )
        );
        $Delete_selectedTheme_Assets = $shop->api()->rest('PUT', "/admin/api/". $version ."/themes/" . $theme_ID . " /assets.json", $array )['body'];
        return json_encode($Delete_selectedTheme_Assets['asset']);
    }
```

### function to Get script tags from Shopify using Script Tag API's
```    
    public function Get__shopify_ScriptTags($shop , $version){
        
        $shop = auth()->user();
        $version = env('SHOPIFY_API_VERSION');
        $user_script_tag = CartButtonhiderDetail::where('user_id',  auth()->user()->id)->first();

        if( !is_null($user_script_tag) ){
            CartButtonhiderDetail::where('user_id', auth()->user()->id)->update(
                array(
                    'status' => 0,
                    'shopify_scripttag_id' => NULL
                )
            );
            // /admin/api/2022-04/script_tags.json
            $script_tag = $shop->api()->rest('GET', "/admin/api/" . $version . "/script_tags.json")['body']['script_tags'];
            return json_encode( $script_tag );
        }else{
            // echo "<pre>";
            return "No Script Tags Found!!!";
            // echo "</pre>";
        }
    }

```

### function to Get all webHooks using Events API's
```
     public function Get__installed_webHooks($shop , $version){
        
        $shop = auth()->user();
        $version = env('SHOPIFY_API_VERSION');
        $user_ = CartButtonhiderDetail::where('user_id',  $shop->id)->first();

        if( !is_null($user_) ){
            // /admin/api/2022-04/webhooks.json
            $webHooks = $shop->api()->rest('GET', "/admin/api/" . $version . "/webhooks.json")['body']['webhooks'];
            // return json_encode( $script_tag );
            return ( $webHooks );
        }else{
            // echo "<pre>";
            return "No WebHooks Found!!!";
            // echo "</pre>";
        }
    }
```

### function to Delete all webHooks using Events->webhooks API's
```
    public function Delete_allInstalled_webHooks($shop , $version){
        
        $shop = auth()->user();
        $version = env('SHOPIFY_API_VERSION');
        $user_ = CartButtonhiderDetail::where('user_id',  $shop->id)->first();

        if( !is_null($user_) ){
            // /admin/api/2022-04/webhooks.json
            $webHooks = $shop->api()->rest('GET', "/admin/api/" . $version . "/webhooks.json")['body']['webhooks'];
            // return json_encode( $script_tag );
            $deleted__webhooksArray = [];
            foreach ($webHooks as $key => $webHook) {
                // echo $webHook['id'];
                $deleted__webhooksArray[] =[ 'webhook_'.$key => $webHook['id'] ];
                if( !is_null($webHook) ){
                    $shop->api()->rest('DELETE', "/admin/api/" . $version . "/webhooks/". $webHook['id'] .".json");
                }
            }
            return ( $deleted__webhooksArray );
        }
    }
    
```

### function to reset/ delete script tags from Shopify using Script Tag API's
```
public function Reset__shopify_ScriptTags($shop , $version){
        
        $shop = auth()->user();
        $version = env('SHOPIFY_API_VERSION');
        $user_script_tag = CartButtonhiderDetail::where('user_id',  auth()->user()->id)->first();

        if( !is_null($user_script_tag) ){
            CartButtonhiderDetail::where('user_id', auth()->user()->id)->update(
                array(
                    'status' => 0,
                    'shopify_scripttag_id' => NULL
                )
            );
            // /admin/api/2022-04/script_tags.json
            $script_tag = $shop->api()->rest('GET', "/admin/api/" . $version . "/script_tags.json")['body']['script_tags'];
            
            foreach ($script_tag as $key => $s_tags_value) {
                // echo $s_tags_value['id'];
                $script_tag = $shop->api()->rest('DELETE', "/admin/api/" . $version . "/script_tags/" . $s_tags_value['id'] . ".json")['body'];
                // echo "<pre>";
                // print_r($script_tag);
                // echo "</pre>";
            }
            return "Done Removing Script Tags from shopify Store.";
        }else{
            // echo "<pre>";
            return "No Script Tags Created!!!";
            // echo "</pre>";
        }
    }
```
