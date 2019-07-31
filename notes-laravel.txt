<?php

// --- CREATING AN API WITH LARAVEL --- //

// install a Laravel installer:

$ composer global require laravel/installer

// if composer isn't installed, install it with $ brew install composer

// only need to run this once.

// to create a new Laravel project:

$ laravel new example-project-name

// once the app is made, the project will be set up. The vendor folder is the equivalent of node modules, we will only really use the folders: routes, app, database.

// we won't use scotchbox, because it's for basic php stuff. Homestead is setup specifically for doing laravel stuff.

// go into the directory, then:

$ composer require laravel/homestead

// sets up homestead.

$ vendor/bin/homestead make

// change Homestead.yaml -> memory: 512

$ vagrant up

// go to http://homestead.test or http://localhost:8000

// if you move directories, change the folders: - map: url part of the Homestead.yaml file.


// --- LARAVEL --- //

// modern php framework. Php first came out in 94, was only a templating language, not even a programming language. It was made so you can have two html files that used the same code without having to copy and paste things around.

// laracasts is better than stack overflow for laravel related issues.

// laravel forge allows you to easily host laravel app. (not free)

// remember to vagrant ssh before running any artisan commands


// Routing //

// same as what routing means in React, mapping pages to urls.

// go to /routes/api.php, delete everything that's already in there, then:


$router->post("/articles", "Articles@store");

// if the user does a post request to /articles, it will call the bit of code called 'Articles@store'

// make a controller: a file that gets called by a route. The controller then tells Laravel what to do.

// above, Articles@store is a controller (articles) and a method(store)

// go into the code directory in the vagrant box, then we can run artisan commands:

$ artisan make:controller Articles --api

// the --api part puts in all the methods that we want to use. This will make a controller file in app/Http/Controllers/ that contains a load of methods (including the store method we used above). The next thing to do is to put code in these methods so that Laravel knows what to do when we call them.

// Database Migrations:

// scripts that set up the database in the same way that it was set up at the original location. They are run by Laravel when we run: $ artisan migrate

// creating a Model/Migration:

$ artisan make:model Article -m

// makes the model class and creates a migration file in the database migrations directory (-m does this)

// the up and down functions in this file allow you to setup and destroy the migrated tables.

// in the up function, $table->increments('id') creates a column, $table->timestamps() makes a created at and updated at column for your table. We also need to add other columns, like title, article, etc. Laravel supports many types of databases, so it uses more generic naming:

$title->string('title', 100); // similar to using VARCHAR in mySQL
$title->text('article'); // no length for text fields.

// this file doesn't give the database any structure, because we haven't run these migrations yet. Can use Sequel Pro to look at a mysql database with a gui. Mark is a command line obsessive, and still prefers a gui, it is much faster, you should do it. To log in:

host: homestead.test // or 127.0.0.1
username: homestead
password: secret
database: homestead
port: 3306 // or 33060

$ artisan migrate

// this command sets up the database to have the correct structure.

// if you shared your code on github now, someone could download it, do artisan migrate and they would have a db with the right structure.

// the migrations table keeps track of which migrations have already run to keep from running duplicate migrations.

// Models:

// Eloquent is the Object Relational Mapper, turns db queries into php classes and methods. We use the Model class to interact with the db:

class Article extends Model
{
  protected $fillable = ["title", "article"];
}

$article->title; // gives us the article title
$article->title = "Blah blah blah"; // set the article title
$artilce->save(); // save it to the db.

// Controller:

// a piece of code that is run for a specific route, whose job it is to get/update the relevant data and return a response to a user.

// put use App\Article; to tell php to import something. This is namespacing.

// store method:

public function store(Request $request) // here, the request is the request the user made.
{
  $data = $request->only(["title", "article"]) // an object with an only method. This only method filters out everything but what you give it from the request.

  $article = Article::create($data); // a static method. This does all the mysql stuff and then stores the response in a variable called $article. Article:: uses the model we made earlier.

  return response($article, 201); // return the result and a 201 (created) status code.
}


can be slimmed down:

public function store(Request $request)
{
  $data = $request->only(["title", "article"]);
  return Article::create($data);
}

// adding get:

// add a route: go to routes/api, add a get, second arg: Articles@index, go into controller, function index(), fill function body with return Article::all();

// grouping router groups:

$router->group(["prefix" => "articles"], function ($router) {
  $router->get("", "Articles@index");
  $router->post("", "Articles@store");
  $router->get("{article}", "Articles@show"); // this creates a url parameter
})

// in controller, go to show:

public function show($id)
{
  return Article::find($id);
}

// to put an article:

$router->put("{article}", "Articles@update");

public function update(Request $request, $id)
{
  // find current article
  $article = Article::find($id);

  // get the requested data
  $data = $request->only(["title", "article"]);

  // update the article
  $article->fill($data)->save();

  // return updated version
  return $article;
}

// to delete:

$router->delete("{article}", "Articles@destroy");

public function destory($id)
{
  $article = Article::find($id);
  $article->delete();

  return response(null, 204); // success, but nothing to give back.
}


// 404s:

// use type hinting. Put the name of the expected argument (in this case, asking for an article):

public function show(Article $article)
{
  return $article;
}

// if the id, coming in as the variable $article, is not valid, Laravel won't find anything in the database, and will return a 404. If it finds it, it will fetch the article, and it will assign that to the variable $article instead of the id that was fed in.


// Resources:

// Resources let us control the format of the JSON output.

$ artisan make:resource ArticleResource

public function toArray($request)
{
  return [
    "id" => $this->id,
    "title" => $this->title,
    "article" => $this->article,
  ];
}

// reference the article resource at the top of the controller:

use App\Http\Resources\ArticleResources

// then use it to return things:

return new ArticleResource($article);

// this also adds a data property to the json result.

// to use a resource on multiple things, you need to use a double colon and 'collection'. No 'new' keyword is needed either:

return ArticleListResource::collection(Article:all());

// CORS:

// Cross-Origin Resource Sharing - stops things being able to use resources owned by the browser when they shouldn't have access. As such, a browser can't access your API unless you set it up to do so. It's complicated, so we just use a library that does it for us:

$ composer require barryvdh/laravel-cors

$ php artisan vendor:publish --provider="Barryvdh\Cors\ServiceProvider"

// makes a new file in config called cors.php.

// in config/app.php, edit providers list:

'providers' => [
    // ...other providers...
    Barryvdh\Cors\ServiceProvider::class,
],

// in app/Http/Kernel.php

protected $middleware = [
    // ...other middleware...
    \Barryvdh\Cors\HandleCors::class,
];

// make sure to put this in the top section of middleware in this file.

// after all this, everything works just the same as it did before in Postman, but now it should work using axios in a browser.



// Validation:

// should always validate data on the server. Cannot trust JS validation, but you should do both. Server validation is the most important, though. JS validation can easily be bypassed, but it is nice for user experience.

// if you don't have validation on the server side, you will get a lot of 500 errors.

// to avoid mysql errors:

// required: db fields that cannot be null should have required validation
// max: make sure that you don't accept anything that is longer than the max field length,
// date / integer / string: check formats before inserting into db

$ artisan make:request ArticleRequest

class ArticleRequest extends FormRequest
{
  public function authorize() // returning false means you can't submit form at all
  {
    return true;
  }

  public function rules()
  {
    return [
      "title" => ["required", "string", "max:100"], //  here, string is very general,
      "article" => ["required", "string", "min:50"], // just string of characters
    ];
  }
}

// look at validation rules on Laravel docs to see all the different ways to validate.

// reference in controller:

use App\Http\Requests\ArticleRequest;

// then just change type hinting:

public function store(ArticleRequest $request) { /* ... */ }

// this takes care of the validation for you, returning a 422 unprocessable entity if the request is not valid (i.e. the user has missed off some fields).


// ONE-to-MANY:

// each article can have MANY comments, but each comment can only have ONE article. How do we store a one to many relationship in a database?

// make another table for the MANY, give them a foreign key that relates to the id of the ONE. The ONE doesn't need to keep track of which of the MANY belong to it.

// graph databases are useful for finding relationships rather than storing relationships. SQL is good for relational data.

$ artisan make:model Comment -m

// make a model called Comment and add -m to make a migration.

// edit migration:

Schema::create('comments', function(Blueprint $table) {
  $table->increments('id');
  $table->timestamps();

  $table->string('email', 254);
  $table->text('comment');

  $table->integer('article_id')->unsigned();
  // the ->unsigned() bit means it doesn't have a sign, i.e. a plus or minus sign, which saves some space in the db, but more importantly, it prevents an error that would pop up in the next stage when we set up the foreign key. It needs to match the type of the column id on the articles table, which is an unsigned integer.

  $table->foreign('article_id')->references('id')->on('articles')->onDelete('cascade');
  // this links the article id to the articles table and sets them to be deleted when the article itself is deleted.
})

$ artisan migrate // to make the comments table

// we need to tell Laravel that an article has comments and that comments belong to an article.

// go to the model classes and add relationships:

class Article extends Model
{
  protected $fillable = ['title', 'article'];

  public function comments()
  {
    return $this->hasMany(Comment::class);
  }

}

class Comment extends Model
{
  protected $fillable = ['email', 'comment'];

  public function article()
  {
    return $this->belongsTo(Article::class);
  }

}

// create route for comments:

$router->post("{article}/comments", "Comments@store");

// create a comments controller:

$ artisan make:controller Comments --api

// edit controller:

use App\Article;
use App\Comment;

public function store(Request $request, Article $article)
{
  $comment = new Comment($request->only(['email', 'comment']));
  // create new comment (not Comment::create(), as this will insert it immediately)
  // store the comments on the article:
  $article->comments()->save($comment); // this uses the function that we made on the article model
  return $comment;
}

// add validation:

$ artisan make:request CommentRequest

public function rules()
{
  return [
    "email" => ["required", "max:254", "email"],
    "comment" => ["required", "string"],
  ]
}

// import into comments controller:

use App\Http\Requests\CommentRequest

public function store(CommentRequest $request, Article $article) {...}

// set up get route:

$router->get("{article}/comments", "Comments@index");

// update controller:

public function index(Article $article)
{
  return $article->comments;
}

// add resource:

$ artisan make:resource CommentResource

// edit comment resource:

return [
  "id" => $this->id,
  "email" => $this->email,
  "comment" => $this->comment,
]

// set up route, go to controller, add validation and resources becomes common.


// MANY-to-MANY relationships:

// can't add the relationship in either table, so we make a pivot table called something like article_tag to link articles and tags. Use table names in alphabetical order in the singular.

// the pivot table has an id for its own entries, and it has the ids for article and for tag. Laravel deals with the pivot table for us. All we have to do is create it in a migration, set up the foreign keys and forget about it largely.

$ artisan make:model Tag -m

// make table schema:

$table->string('tag', 50);

// delete timestamps if they're not necessary.

// can make more than one table in the public function up()

// if you make more than one table in the up, you need to get rid of both in the down. Delete them in the order that deletes reliant tables before relied upon tables so that there is never a moment when there is a table that relies on something that is deleted.

// update the article model to have a relationship to tags, and the tags model to have a relationship to tags.

public function tags()
{
    return $this->belongsToMany(Tag::class);
}

class Tag extends Model
{
    public $timestamps = false;
    protected $fillable = ['name'];

    public function articles()
    {
        return $this->belongsToMany(Article::class);
    }

}

// seeing as tag model doesn't use timestamps, you need to set public $timestamps = false;

// update ArticleRequests:

'tags' => ['required', 'array'],
'tags.*' => ['required', 'string'],

// update Articles controller:

$tags = Tag::parse($request->get('tags'));
$article->setTags($tags);

// add parse static method to Tag model:

class Tag extends Model
{
  public static function parse(array $tags)
  {
    return collect($tag)->function ($tag)
    {
      $string = trim($tag);

      return static::makeTag($string);
    }
  }

  private static function makeTag($string)
  {
    $exists = Tag::where('name', $string)->first();

    return $exists ?: Tag::create(['name' => $string]);
  }
  /* ... */
}

// use collect to make the array into a collection, which makes it much more like in JS

// add setTags method (nonstatic) to Article model:

use Illuminate\Support\Collection;

public function setTags(Collection $tags)
{
  // update the pivot table with tag IDs:
  $this->tags()->sync($tags->pluck('id')->all());
  return $this
}

// update ArticleResource:

return [
  /* ... */
  "tags" => $this->tags->pluck('name'),
]

// --- Building an app --- //

// Planning:
// plan mvp
// plan wireframes
// plan routes
// plan database structure
// plan redux actions

// --- Cloning Laravel --- //

// - clone project, cd into directory
// - make sure composer is installed
// - $ composer install
// - $ vendor/bin/homestead make
// - $ mv .env.example .env
// - $ vagrant up
// - $ vagrant ssh
// - $ cd code
// - $ art key:generate
// - $ art migrate
// - $ exit
// - $ npm install
// - $ npm run watch
