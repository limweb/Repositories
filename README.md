# Laravel Repositories

Laravel Repositories is a package for Laravel 5 which is used to abstract the database layer. This makes applications much easier to maintain.

## Installation

Run the following command from you terminal:


 ```
 composer require "bosnadev/repositories: 0.*"
 ```
 
or add this to require section in your composer.json file:
 
 ```
 "bosnadev/repositories": "0.*"
 ```
 
then run ```composer update```
 
 
## Usage
 
First, create your repository class. Note that your repository class MUST extend ```Bosnadev\Repositories\Eloquent\Repository``` and implement model() method

    <?php namespace App\Repositories;

    use Bosnadev\Repositories\Contracts\RepositoryInterface;
    use Bosnadev\Repositories\Eloquent\Repository;

    class FilmsRepository extends Repository {

        public function model() {
            return 'App\Film';
        }
    }

By implementing ```model()``` method you telling repository what model class you want to use. Now, create ```App\Film``` model:

    <?php namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Film extends Model {
    
        protected $id = 'film_id';
    
        protected $table = 'film';
    
        protected $casts = [
            "rental_rate"       => 'float'
        ];
    }
    
And finally, use the repository in the controller:

    <?php namespace App\Http\Controllers;
    
    use App\Repositories\FilmsRepository as Film;
    
    class FilmsController extends Controller {
    
        private $film;
    
        public function __construct(Film $film) {
    
            $this->film = $film;
        }
    
        public function index() {
            return \Response::json($this->film->all());
        }
    }
    

## Available Methods

The following methods are available:

##### Bosnadev\Repositories\Contracts\RepositoryInterface

* public function all($columns = array('*'))
* public function paginate($perPage = 1, $columns = array('*'));
* public function create(array $data)
* public function update(array $data, $id)
* public function delete($id)
* public function find($id, $columns = array('*'))
* public function findBy($field, $value, $columns = array('*'))

##### Bosnadev\Repositories\Contracts\CriteriaInterface

* public function apply($model, Repository $repository)


### Example usage


Create a new film in repository:

    $this->film->create(Input::all());
    
Update existing film:

    $this->film->update(Input::all(), $film_id);
    
Delete film:
    
    $this->film->delete($id);
    
Find film by film_id;

    $this->film->find($id);
    
you can also chose what columns to fetch:
   
    $this->film->find($id, ['title', 'description', 'release_date']);
    
## Criteria

Criteria is a simple way to apply specific condition, or set of conditions to the repository query. Your criteria class MUST implement ```Bosnadev\Repositories\Contracts\CriteriaInterface``` interface.

Here is a simple criteria:

    <?php namespace App\Repositories\Criteria\Films;
    
    use Bosnadev\Repositories\Contracts\CriteriaInterface;
    use Bosnadev\Repositories\Contracts\RepositoryInterface as Repository;
    use Bosnadev\Repositories\Contracts\RepositoryInterface;
    
    class LengthOverTwoHours implements CriteriaInterface {
    
        /**
         * @param $model
         * @param RepositoryInterface $repository
         * @return mixed
         */
        public function apply($model, Repository $repository)
        {
            $model = $model->where('length', '>', 120);
            return $model;
        }
    }
    
Now, inside you controller class you call pushCriteria method:

    <?php namespace App\Http\Controllers;
    
    use App\Repositories\Criteria\Films\LengthOverTwoHours;
    use App\Repositories\FilmsRepository as Film;
    
    class FilmsController extends Controller {
    
        /**
         * @var Film
         */
        private $film;
    
        public function __construct(Film $film) {
    
            $this->film = $film;
        }
    
        public function index() {
            $this->film->pushCriteria(new LengthOverTwoHours());
            return \Response::json($this->film->all());
        }
    }
