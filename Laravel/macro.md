# Eloquent: Collections

Here's how our macro could look like. You can put it the boot method of App\Providers\AppServiceProvider or a service provider of your own.

    Builder::macro('whereLike', function ($attributes, string $searchTerm) {
        $this->where(function (Builder $query) use ($attributes, $searchTerm) {
            foreach ($attributes as $attribute) {
                $query->orWhere($attribute, 'LIKE', "%{$searchTerm}%");
            }
        });

        return $this;
    });

call this code any model by this 
> `$medicineLists = MedicineDetails::whereLike(['brand_name', 'manufacturer_name'], $request->search)->paginate(2);`

# Adding support for relations : Macro
As it stand we can now only search the attributes of the model where using the scope on. Let's add support for searching the attributes of the relations of that model as well.

    Builder::macro('whereLike', function ($attributes, string $searchTerm) {
        $this->where(function (Builder $query) use ($attributes, $searchTerm) {
            foreach (array_wrap($attributes) as $attribute) {
                $query->when(
                    str_contains($attribute, '.'),
                    function (Builder $query) use ($attribute, $searchTerm) {
                        [$relationName, $relationAttribute] = explode('.', $attribute);

                        $query->orWhereHas($relationName, function (Builder $query) use ($relationAttribute, $searchTerm) {
                            $query->where($relationAttribute, 'LIKE', "%{$searchTerm}%");
                        });
                    },
                    function (Builder $query) use ($attribute, $searchTerm) {
                        $query->orWhere($attribute, 'LIKE', "%{$searchTerm}%");
                    }
                );
            }
        });

        return $this;
    });

With that macro can do something like this:

>`Post::whereLike(['name', 'text', 'author.name', 'tags.name'], $searchTerm)->get();`
