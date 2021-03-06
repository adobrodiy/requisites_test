# requisites
Пакет для генерации и валидации тестовых ОГРН

Подключаем
```javascript
const generatorFactory = require( 'generator' );
```

Конфигурируем
```javascript
const generator = generatorFactory( null, {
	initVals: [ "1", "5" ]
} );
```

Генерируем
```javascript
const ogrn = generator.generate();
```

Валидируем
```javascript
const validation = generator.validate( ogrn );
const valid = Object.keys( validation ).reduce( ( valid, key ) => {
    return valid && validation[ key ];
}, true );

console.log( `ogrn is valid: ${ valid }` );
```

## конфигурация
### generatorFacory( [ defaultConfigOptions [, config ]] )
generatorFactory принимает два необязательных параметра

#### defaultConfigOptions
Объект, свойства которого используются для конфигурации дефолтного config. Если не используются дефолтный config, не используются defaultConfigOptions.

```javascript
{
	yearsLength : 20,
	codesCount  : 20,
	numbersCount: 20
}
```

###### yearsLength
По умолчанию - 20. Число последних лет, для которых генерируется ОГРН. Например, если текущий год - 2017, и `defaultConfigOptions.yearsLength == 3`, то ОГРН будет генерироваться для 2017, 2016, 2015 годов - `[ "17", "16", "15" ]`. Таким образом, это настройка для `config.years` по умолчанию. [Подробнее о config.years](#years)

###### codesCount
По умолчанию - 20. Число случайно сгенерированных кодов налоговых инспекций, из которых будет случайно выбираться код при генерации ОГРН, а также при валидации ОГРН код будет проверяться на вхождение в этот список. Например, если `defaultConfigOptions.codesCount = 3`, то сгенерируется 3 кода налоговых испекций. Например `[ "12", "34", "56" ]`. Таким образом, это  настройка для `config.codes` по умолчанию. [Подробнее о config.codes](#codes)

###### numbersCount
По умолчанию - 20. Число случайно сгенерированных номеров записей ОГРН, из которых будет случайно выбираться номер записи при генерации ОГРН, а также при валидации ОГРН номер записи будет проверяться на вхождение в этот список. Например, если `defaultConfigOptions.numbersCount = 3`, то сгенерируется 3 номера записи ОГРН. Например `[ "12345", "67890", "48162" ]`. Такми образом, это настройка для `config.numbers` по умолчанию. [Подробнее о config.numbers](#numbers)

#### config
Объект, свойства которого содержат исходные данных из которых генерируется ОГРН, и которые используются для валидации ОГРН.

```javascript
{
	initVals: ...,
	years: ...,
	subjects: ...,
	codes: ...,
	numbers: ...	
}
```

###### initVals
Набор первых знаков. Массив строк из одной цифры.

По умолчанию - массив `[ "1", "5" ]`.

При генерации строки ОГРН 1-й знак будет соответсвовать одному из значений массива первых знаков. При валидации поле `year` валидационного объекта получит значение `true`, только если 1-й знак будет соответствовать одному из значений массива первых знаков.

###### years
Набор годов внесения ОГРН в реестр. Массив строк из двух цифр.

По умолчанию - массив строк из двух цифр, длина которого `defaultConfigOptions.yearsLength`. Строки соответствуют последним годам - текущему и предшествующим. Например, если текущий год - 2017, и ` defaultConfigOptions.yearsLength == 3`, то по умолчанию `config.years = [ "17", "16", "15" ]`. [Подробнее о defaultConfigOptions.yearsLength](#yearslength)

При генерации строки ОГРН 2-й и 3-й знаки вместе будут соответсвовать одному из значений массива годов внесени ОГРН в реестр. При валидации поле `year` валидационного объекта получит значение `true`, только если 2-й и 3-й знаки вместе будут соответствовать одному из значений массива годов внесения ОГРН в реестр.

###### subjects
Набор номеров субъектов РФ. Массив строк из двух цифр.

По умолчанию - массив из файла src/lib/subjects.json такого вида `[ "01", "02", ... ]`.

При генерации строки ОГРН 4-й и 5-й знаки вместе будут соответствовать одному из значений массива номеров субъектов РФ. При валидации поле `subject` валидационного объекта получит значение `true`, только если 4-й и 5-й знаки будут соответствовать одному из значений массива номеров субъектов РФ.

###### codes
Набор кодов налоговых инспекций. Либо массив строк из двух цифр. Либо объект, группирующий массивы строк из двух цифр по объектам РФ.
То есть `codes` может иметь такой вид:
```javascript
const config = {
	...
	codes: [ "04", "08", "15", "16", "23", "42" ],
	...
}
```
В этом случае при генерации строки ОГРН 6-й и 7-й знаки вместе будут соответсвовать одному из элементов массива кодов налоговых инспекций `config.codes`. При валидации поле `code` валидациооного объекта получит значение `true`, только если 6-й и 7-й знак валидируемой строки вместе будут соответсвовать одному из значений массива кодов налоговых инспекций `config.codes`.

Также `codes` может иметь вот такой вид: 
```javascript
const config = {
	...
	codes: {
		"01": [ "04", "08", "15" ],
		"02": [ "16", "23", "42" ]
	},
	...
};
```

В этом случае при генерации строки ОГРН 6-й и 7-й знаки вместе будут соответсвовать одному из элементов массива кодов налоговых инспекций cубъекта РФ `config.codes[subject]`. То есть сначала случайным образом из `config.subjects` будет выбран субъект РФ `subject` и подставлен на место 4-го и 5-го знаков генерируемой строки. Затем субъект РФ `subject` будет использован как ключ, по которому из группированного по субъектам РФ набора массивов кодов налоговых инспекций `config.codes` будет выбран массив кодов налоговых инспекций соответствующего субъекта РФ `config.codes[subject]`. Затем из массива случайным образом будет выбран код налоговой инспекции, которому будут соответсвовать 6-й и 7-й знаки генерируемой строки ОГРН. 

При валидации поле `code` валидациооного объекта получит значение `true`, только если 6-й и 7-й знак валидируемой строки вместе будут соответсвовать одному из элементов массива кодов налоговых инспекций субъекта РФ `config.codes[subject]`. То есть сначала будут выбраны 4-й и 5-й знаки валидируемой строки, они будут восприниматься как номер субъекта РФ `subject`. Затем этот номер будет использован как ключ, по которому будет выбран массив кодов налоговых инспекций `config.codes[subject]`. Если 6-й и 7-й знаки валидируемой строки вместе будут совпадать с одним из элементов выбранного массива, то поле `code` валидационного объекта получит значение `true`. В остальных случаях оно получит значение `false`.

По умолчанию - массив случайно сгенерированных строк из двух цифр, длина которого `defaultConfigOptions.codesCount`.
Например, если `defaultConfigOptionsюcodesCount = 3`, то сгенерируется массив из трех строк. Например `[ "12", "34", "56" ]`. [Подробнее о defaultConfigOptions.codesCount](#codescount)

###### numbers
Набор номеров записей ОГРН. Массив строк из 5 цифр.

По умолчанию - массив строк их 5 цифр, длина которого равна `defaultConfigOptions.numbersCount`.
Например, если `defaultConfigOptions.numbersCount = 3`, то сгенерируется 3 номера записи ОГРН. Например `[ "12345", "67890", "48162" ]`. [Подробнее о defaultConfigOptions.numbersCount](#numberscount)

При генерации ОГРН с 8-го по 12-й знак вместе будут соответсвовать одному из элементов массива номеров записей. При валидации поле `number` валидациооного объекта получит значение `true`, только если c 8-го по 12-й знаки вместе будут соответсвовать одному из эдементов массива номеров записей.


## generator
generateFactory возвращает объект с двумя методами

### generate()
Возвращает строку из 13 цифр - ОГРН

#### 1-й знак
Первый знак. Выбирается случайным образом из элементов `config.initVals`. [Подробнее о config.initVals](#initvals)

#### Со 2-го по 3-й знак
Год выдачи ОГРН. Выбирается случайным образом из элементов `config.years`. [Подробнее о config.years](#years)

#### C 4-го по 5-й знак
Порядковы номер субъекта РФ. Выбирается случайным образом из элементов `config.subjects`. [Подробнее о config.subjects](#subjects)

#### C 6-го по 7-й знак
Код налоговой инспекции. Выбирается случайным образом из элементов `config.codes`. [Подробнее о config.codes](#codes)

#### C 8-го по 12-й знак
Номер записи, внесенной в реестр в течение года. Выбирается случайным образом из элементов `config.numbers`. [Подробнее о config.numbers](#numbers)

#### 13-й знак
Контрольное число. Младший разряд остатка от деления предыдущего 12-значного числа на 11, если остаток от деления равен 10, то контрольное число равно 0 (нулю).

### validate( ogrnStr )
Валидирует строку `ogrnStr` на соответствие исходным данным `config`. Возвращает валидационный объект `validatation`, поля которого получают boolean значения.

#### validatation ( валидационный объект )
Объект имеет следующие поля

###### lengthVal
Получает значение `true`, если длина валидируемой строки 13. В остальных случаях получает значение `false`.

###### initVal

###### year
Получает значение `true`, если 2-й и 3-й знаки валидируемой строки `ogrnStr` вместе совпадают с одним из элементов `config.years`. [Подробнее о config.years](#years). В остальных случаях получает значение `false`.

###### subject
Получает значение `true`, если 4-й и 5-й знаки валидируемой строки `ogrnStr` вместе совпадают с одним из элементов `config.subjects`. [Подробнее о config.subjects](#subjects). В остальных случаях получает значение `false`.

###### code
Получает значение `true`, если 6-й и 7-й знаки валидируемой строки `ogrnStr` вместе совпадают с одним из элементов `config.codes`. [Подробнее о config.codes](#codes). В остальных случаях получает значение `false`.

###### number
Получает значение `true`, если с 8-го по 12-й знаки валидируемой строки `ogrnStr` вместе совпадают с одним из элементов `config.numbers`. [Подробнее о config.numbers](#numbers). В остальных случаях получает значение `false`.

###### check
Получает значение `true`, если контрольное число равное младшему разряду остатка от деления предыдущего 12-значного числа на 11 совпадает с 13-м знаком валидируемой строки `ogrnStr` ( если остаток от деления равен 10, то контрольное число равно 0). В остальных случаях получает значение `false`.
