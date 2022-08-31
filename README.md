# Laravel-copy-table-data-to-another-table-and-remove-duplicate-data
laravel copy data from one table to another and prevent[remove] duplicate data from database table
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {

//-------------------------------------------Copy data one table data to another table----------------------------
            Client::query()
                        ->where('updated_at', '<', now()->subminutes(1))
                        ->each(function ($oldPost) {
                            $newPost = $oldPost->replicate();
                            $newPost->setTable('newclients');
                            $newPost->save();
                            //$newPost->delete();
                        });

            //DB::table('newclients')->where('name', 'Faysal')->delete();

//-------------------------------------------delete the duplicate data from table-----------------------

// Get all duplicated values. Replace 'table' and 'name' accordingly
$duplicates = DB::table('newclients') // replace table by the table name where you want to search for duplicated values
              ->select('id', 'name') // name is the column name with duplicated values
              ->whereIn('name', function ($q){
                $q->select('name')
                ->from('newclients')
                ->groupBy('name')
                ->havingRaw('COUNT(*) > 1');
              })
              ->orderBy('name')
              ->orderBy('id') // keep smaller id (older), to keep biggest id (younger) replace with this ->orderBy('id', 'desc')
              ->get();

$value = "";

// loop throuht results and keep first duplicated value
foreach ($duplicates as $duplicate) {
  if($duplicate->name === $value)
  {
    DB::table('newclients')->where('id', $duplicate->id)->delete(); // comment out this line the first time to check what will be deleted and keeped
    echo "$duplicate->name with id $duplicate->id deleted! \n";
  }
  else
    echo "$duplicate->name with id $duplicate->id keeped \n";
  $value = $duplicate->name;
}





//------------------------ store data into table---------------------------

$client = new Client();

$client->name = $request->has('cname')? $request->get('cname'):'';
$client->save();
return back();



    }
