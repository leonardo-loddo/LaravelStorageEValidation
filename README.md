Implementare:

Salvataggio nel DB di un'immagine

Validation Rules

aggiungo il return redirect alla funzione store in modo da rimandare agli articoli
    return redirect()->route('article.index');

aggiungo la logica per mostrare il messaggio di successo
    return redirect()->route('article.index')->with('success', 'Libro Caricato');

nel layout creo un if per mostrare il messaggio di successo nel caso si venisse rendirizzati dalla store

    @if(session('success'))
            <div role="alert" class="alert alert-success">
                {{session('success')}}
            </div>
    @endif

sotto il csrf token nella create aggiungo
    @method('POST')

aggiungo preventivamente anche quella per gli errori
    @if(session('success'))
        <div role="alert" class="alert alert-success">
            {{session('success')}}
        </div>
    @endif
    @if(session('errors'))
        <div role="alert" class="alert alert-danger">
            {{session('errors')}}
        </div>
    @endif

creo la rotta parametrica show PER IL DETTAGLIO DI UN ARTICOLO
    Route::get('/article/{article}/show', [ArticleController::class, 'show'])->name('article.show');

nella card aggiungo come href la rotta parametrica per il dettaglio di quell'articolo
    <a href="{{route('article.show', ['article' => $item['id']])}}" class="btn btn-primary">Leggi {{$item->title}}</a>

definisco il metodo show nel controller
    public function show($article){
        $article = Article::find($article); 
        return view('article.show', compact('article'));
    }

aggiungo la logica nel caso a quell'id non corrispondesse nulla
    public function show($article){
        $article = Article::find($article);
        if (!$article) {
            abort(404);
        }
        return view('article.show', compact('article'));
    }

utilizzo una sintassi alternativa nella show per la gestione del 404
    public function show($article){
        $article = Article::findOrFail($article);
        return view('article.show', compact('article'));
    }

metodo migliore con la injection
    public function show(Article $article){
        return view('article.show', compact('article'));
    }

richiamo la card nella vista show
    <section>
        <x-card :item="$article"/>
    </section>

aggiungo la logica di validazione nella store
    public function store(Request $request){
        $request->validate([
            'title' => 'required',
            'body' => 'required',
        ]);
        Article::create([
            'title' => $request->title,
            'body' => $request->body,
        ]);

per questione di ordine come per le funzioni delle rotte, é meglio gestire le logiche di validazione in un posto dedicato

lancio php artisan make:request ArticleStoreRequest

il parametro della store non sará piú un oggetto di tipo Request ma di tipo ArticleStoreRequest
    public function store(ArticleStoreRequest $request){

sposteró le regole di validazione in ArticleStireRequest.php
    public function rules(): array
    {
        return [
            'title' => 'required',
            'body' => 'required',
        ];
    }

elimino la validazione nella store

voglio aggiungere le immagini agli articoli nel mio progetto quindi lancio
php artisan make:migration add_to_articles_table
in modo da aggiungere una colonna a questa tabella del database

riempio le funzioni up e down della migrazione
    public function up(): void
    {
        Schema::table('articles', function (Blueprint $table) {
            $table->string('image')->nullable();
        });
    }

    public function down(): void
    {
        Schema::table('articles', function (Blueprint $table) {
            $table->dropColumn(['image']);
        });
    }

nel modello Article.php aggiungo image al fillable
    protected $fillable = [
        'title', 'body', 'image'
    ];

aggiungo l'input copertina al mio form
    <div class="mb-3">
        <label for="image" class="form-label">Copertina</label>
        <input type="file" class="form-control" name="image">
    </div>

aggiungo l'attributo enctype al mio form per supportare il caricamento di file
    <form action="{{route('article.store')}}" method="POST" enctype="multipart/form-data">

nella store recupero e salvo in due variabili nome dell'immagine e estensione di essa
    public function store(ArticleStoreRequest $request){
        $file_name = $request->file('image')->getClientOriginalName();
        $extension_name = $request->file('image')->getClientOriginalExtension();

modifico nuovamente la funzione store
    public function store(ArticleStoreRequest $request){
        $path_image = '';
        if ($request->hasFile('image')){      //nel caso fosse imessa una immagine
            $file_name = $request->file('image')->getClientOriginalName();  //prendo il suo nome
            $path_image = $request->file('image')->storeAs('public/image', $file_name); //creo una variabile combinando path del file e nome, salvo l'immagine nel server
        }
        Article::create([
            'title' => $request->title,
            'body' => $request->body,
            'image' => $path_image,  //inserisco il path dell'immagine nel server all'interno del mio database
        ]);

nelle regole di validazione aggiungo le estensioni consentite per l'immagine e la grandezza massima di due mega
    public function rules(): array
    {
        return [
            'title' => 'required',
            'body' => 'required',
            'image' => 'mimes:jpg,jpeg,bmp,png|max:2048',
        ];
    }
nella funzione authorize inserisco true come return

richiamo all'interno della card l'immagine
    <img src="{{Storage::url($item->image)}}" class="card-img-top" alt="...">

lancio 
php artisan storage:link
per creare in punlic un collegamento con lo storage