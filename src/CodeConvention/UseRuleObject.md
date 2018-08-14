## 使用规则对象实现超级复杂的验证规则

有的时候由于业务需求的需要，有的字段验证规则比较复杂，比如不简简单单依赖于当前字段值来校验参数是否合法可能还需要依赖请求中的其他字段值去数据库查询后才能够判定当前字段值是否合法，
针对这种对单字段复杂的验证规则我们应该使用规则对象，而不是把这些复杂的查询验证写在控制器或者其他地方。
```
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class ValidElementRule implements Rule
{

    /**
     * Create a new rule instance.
     * @param Illuminate\Request $request
     * @return void
     */
    public function __construct($request)
    {
        $this->request = $request;
    }


    /**
     * 判断验证规则是否通过。
     *
     * @param  string  $attribute
     * @param  mixed  $value
     * @return bool
     */
    public function passes($attribute, $value)
    {
        //some complicated judgement
        if ($value == $something && $this->request->input('key') {
            // if element is not valid then return false
            return false;
        }
        
        return true
    }

    /**
     * 获取验证错误信息。
     *
     * @return string
     */
    public function message()
    {
        return 'The :attribute must be ......';
    }
}
```

一旦规则对象被定义好后，你可以通过将规则对象的实例传递给其他验证规则来将其附加到验证器：

```

<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use App\Rules\ValidElementRule;
class UpdateElementRequest extends FormRequest
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
            'element' => ['required', new ValidElementRule($this)]
        ];
    }
}

```
