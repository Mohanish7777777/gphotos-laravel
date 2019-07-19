# Google Photos API for laravel

[![Build Status](https://travis-ci.com/Tiththa/gphotos-laravel.svg?token=ipyLe4fTTgA1E3pc5jHb&branch=master)](https://travis-ci.com/Tiththa/gphotos-laravel)

https://developers.google.com/photos/

## Requirements
- PHP >= 7.1.3
- Laravel >= 5.8

## Installation

```
composer require revolution/laravel-google-photos
```

This package depends on

- Socialite https://github.com/laravel/socialite
- https://github.com/google/google-api-php-client
- https://github.com/pulkitjalan/google-apiclient

Google_Service_PhotosLibrary  
https://github.com/google/google-api-php-client-services/tree/master/src/Google/Service/PhotosLibrary

### Get API Credentials
from https://developers.google.com/console  
Enable `Photos Library API`.

### publish config file
```
php artisan vendor:publish --provider="PulkitJalan\Google\GoogleServiceProvider" --tag="config"
```

### config/google.php
```php
    'client_id'        => env('GOOGLE_CLIENT_ID', ''),
    'client_secret'    => env('GOOGLE_CLIENT_SECRET', ''),
    'redirect_uri'     => env('GOOGLE_REDIRECT', ''),
    'scopes'           => [\Google_Service_PhotosLibrary::PHOTOSLIBRARY],
    'access_type'      => 'offline',
    'approval_prompt'  => 'force',
    'prompt'           => 'consent', //"none", "consent", "select_account" default:none
```

Google Photos API does not support Service Account.

### config/service.php for Socialite

```php
    'google' => [
        'client_id'     => env('GOOGLE_CLIENT_ID', ''),
        'client_secret' => env('GOOGLE_CLIENT_SECRET', ''),
        'redirect'      => env('GOOGLE_REDIRECT', ''),  //http://yourdomain/callback
    ],
```



### Configure .env as needed
```
GOOGLE_APPLICATION_NAME=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT=
```

### Usage
```php
		$media = $request->user()->photos()->media($id); //show media object
		$optParams = [];

        //        $mediatypefilter = new \Google_Service_PhotosLibrary_MediaTypeFilter();
        //        $mediatypefilter->setMediaTypes('ALL_MEDIA');
        //
        //        $filters = new \Google_Service_PhotosLibrary_Filters();
        //
        //        $filters->setMediaTypeFilter($mediatypefilter);
        //
        //        $optParams = ['pageSize' => 100, 'filters' => $filters];


        // Facade
        $user = $request->user();

        $token = [
            'access_token'  => $user->access_token,
            'refresh_token' => $user->refresh_token,
            'expires_in'    => $user->expires_in,
            'created'       => $user->updated_at->getTimestamp(),
        ];

        $media_object = Photos::setAccessToken($token)->search($optParams);


        // Trait
        //        $media_object = $request->user()->photos()->search($optParams);

        //        dd($media_object);

        $mediaitems = $media_object->mediaItems ?? [];

        //        $nextPageToken = $media_object->nextPageToken ?? null;


        //uploading
        $photos = $request->user()->photos();

        $file = $request->file('file');

        $uploadToken = $photos->upload(
            $file->getClientOriginalName(),
            fopen($file->getRealPath(), 'r')
        );

        $result = $photos->batchCreate([$uploadToken]);

        //Albums

        //        $token = $request->user()->access_token;
        //
        //        Google::setAccessToken($token);
        //
        //        $photos = Google::make('PhotosLibrary');

        $optParams = ['pageSize' => 10];

        // Google's Client Library
        //        $albums_object = $photos->albums->listAlbums($optParams)->toSimpleObject();

        // Facade
        //        $albums_object = Photos::setService($photos)->listAlbums($optParams);

        // PhotosLibrary Trait
        $albums_object = $request->user()->photos()->listAlbums($optParams);

        //        dd($albums_object);

        $albums = $albums_object->albums ?? [];
        $nextPageToken = $albums_object->nextPageToken ?? null;
```






