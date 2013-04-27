Google Doc Exporter
===============
This PHP class let us extract title and content from Google Docs.

Google Doc Exporter aims to provide the following situations at work environment especially at the ones using CMS:

* Separation of content entry work and development work.
* Clients can start content entry work in early stage.
* Developers can continue working on developemnt environment.
* Developers can avoid deleting content data permanently when applying changes from develpoment environment.

Demo site
---------------
[See Demo](http://public.hironozu.com/google_doc_exporter_demo/)

Notice
---------------
* Google Doc Exporter requires PHP 5.3.X or newer.
* Google Doc Exporter just provides the basic feature so you need to integrate this class to your situation.
* This project may become combined into a new project as that new project supports Evernote or something else that I can see as a good storage for content entry work.

Preparation
---------------
You need Google account to use this class.

You also need a setup at your Google API Access settings page:
* https://code.google.com/apis/console/

Screenshots for settings:
* http://note.io/XUPVkJ
* http://note.io/YWbjvY

How to create a document for this class (Specifications)
---------------
* This class provides three ways to get Google Doc. By File ID, Document Title and Download URL. If you use Document Title, the tiles have to be unique to enable to specify the individual document. Therefore, it is recommended to create Google account per web site and use unique title (e.g. "Contact details at upper right area" instead of "Contact").
* Due to the specification above, actual title is extracted from document body, not from document title. To set the title you put text and set as "Title" with "Styles" dropdown list.
* "Heading X" in "Styles" will be Hx tag (e.g. "Heading 1" becomes H1 tag).
* List is replicated in tag level (OL or UL).
* Font size, colour, bullet style for list and other styles are not replicated as Google Doc Exporter is basically aiming to replicate the content and its structure, not appearance (If you need to replicate the appearance, you should just embed the document into the page, that lacks of SEO (doesn't it?)).

TODO (Issues)
---------------
* To extract URL for images to let you save them within your web system.
* To enable to use memcached/session/database to store document data...or not as it can be implemented by individual system.
* Handle error properly? https://developers.google.com/drive/handle-errors
* To provide Joomla extension and Drupal module using this class.

Sample Code
---------------


    session_start();

    require_once 'google_doc_exporter.php';

    // Change the following to suit to your environment
    $exporter = new google_doc_exporter(array(
        'clientId' => '123456789012-abcdefghijklmnopqrstuvwxyz.apps.googleusercontent.com',
        'clientSecret' => '12345678901234567890',
        'redirectUri' => 'http://' . $_SERVER['HTTP_HOST'] . $_SERVER['PHP_SELF'],
    ));

    // This example uses session but the value would be from database in real use
    if ( ! isset($_SESSION['accessToken'])) $_SESSION['accessToken'] = false;
    $accessToken =& $_SESSION['accessToken'];

    if ($accessToken) {
        // Set Access Token to avoid getting authenticated again
        $exporter->setAccessToken($accessToken);
    } else {
        // This redirects to Google if you don't have Access Token and
        // it comes back with Authentication Code to get Access Token which
        // you must store Access Token to avoid getting authenticated again in real use
        $exporter->authenticate($accessToken);
        // Redirect to clear up URL having the query "code"
        $exporter->redirect();
    }
    // var_dump($accessToken);

    // To get all the document data
    $listOfFiles = $exporter->getList();
    // var_dump($listOfFiles);

    $fileId = $listOfFiles[0]['id'];
    $downloadUrl = $listOfFiles[0]['exportLinks']['text/html'];

    // Get content by ID
    $content = $exporter->getContentById($fileId);
    // This way uses inline CSS for STRONG, EM and U tag
    $content = $exporter->getContentById($fileId, array('inline' => true));
    // You can actually use document title as title if you prefer
    // $content['title'] = $listOfFiles[0]['title'];

    // Get content by download URL
    $content = $exporter->getContentByUrl($downloadUrl);
    // This way uses inline CSS for STRONG, EM and U tag
    $content = $exporter->getContentByUrl($downloadUrl, array('inline' => true));

    // Get content by title
    $content = $exporter->getContentByTitle('Test');
    // This way uses inline CSS for STRONG, EM and U tag
    $content = $exporter->getContentByTitle('Test', array('inline' => 1));

    ?>
    <!doctype html>
    <html>
    <head>
    <meta charset="utf-8">
    </head>
    <body>
    <header><h1><?php echo $content['title']; ?></h1></header>
    <div class="content">
        <?php echo $content['body']; ?>
    </div>
    </body>
    </html>

