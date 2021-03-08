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