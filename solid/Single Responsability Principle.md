# Single Responsability Principle

<p align="center">
   "every entity, every class or every method should do only one job or have only one reason to change"
</p> 

---
Com essa frase podemos chegar a conclusão que toda entidade, toda classe ou todo método deve fazer apenas uma tarefa, ter apenas uma responsabilidade, apenas uma razão para ser modificado.

O melhor exemplo pra mim é no contexto do Controller do laravel, na qual é uma classe que deve ser responsável apenas em gerenciar o request e o response. Logo, fazer várias tarefas em cima dessa classe acaba fugindo do princípio de responsabilidade única. Por exemplo, regras de negócio, chamadas no banco, validações, precisam estar em classes separadas.

Vamos usar o método store do controller como exemplo:
```php
class UserController extends Controller
{
    public function create()
    {
        return view('user.create');
    }
    public function store(Request $request)
    {
        $validated = request()->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|max:80',
            'mobile' => 'required|digits:11',
            'address1' => 'required|email|max:80',
            'postcode' => 'required|digits:11',
            'state' => 'required|string|in:ES,RJ,SP,MG',
            'street' => 'required|string|max:80',
            'suburb' => 'required|string|max:80',
            'apt_number' => 'string|numeric|required_if:house_number,null',
            'house_number' => 'string|numeric|required_if:apt_number,null',
        ]);
        User::create($validated);
        return  redirect()->route('index');
    }
}
```
O código acima fere o principio de responsabilidade única, já que o controller está carregado de uma tarefa na qual não é de sua reponsabilidade, ou seja, a validação tem que ocorrer em uma classe responsável pelo request. Como exemplo, usaremos a classe StoreUserRequest extendendo o FormRequest, que é uma classe de requests personalizada que encapsula sua própria lógica de validação e autorização.
```php
namespace App\Http\Requests;
use Illuminate\Foundation\Http\FormRequest;
class StoreUserRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [ 
            'name' => 'required|string|max:255',
            'email' => 'required|email|max:80',
            'mobile' => 'required|digits:11',
            'address1' => 'required|email|max:80',
            'postcode' => 'required|digits:11',
            'state' => 'required|string|in:ES,RJ,SP,MG',
            'street' => 'required|string|max:80',
            'suburb' => 'required|string|max:80',
            'apt_number' => 'string|numeric|required_if:house_number,null',
            'house_number' => 'string|numeric|required_if:apt_number,null',
         ];
    }
}
```
Já com a classe StoreUserRequest criada, teremos o seguinte controller.

```php
class UserController extends Controller
{
    public function create()
    {
        return view('user.create');
    }

    // Registra o usuario no banco de dados
    public function store(StoreUserRequest $request)
    {
        // Os valores dentro da request já estão validados nesse ponto
        $validated_with_role = $request->validated()
        User::create($validated_with_role);
        return  redirect()->route('index');
    }
}
```
Com isso, transferimos a tarefa de validação do request para a classe responsável e respeitamos o princípio de Responsabilidade única, deixando o método store muito mais simples e fazendo somente a tarefa de receber a solicitação e retornar a resposta.
