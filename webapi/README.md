Use this sample to test connection from a web app to SQL DB. It uses Entity Framework code first. To run the app for the first time:

1. Configure Connection string in appSettings.json
2. Run `dotnet ef migrations add InitialCreate`
3. Run `dotnet ef database update`
4. Run or deploy the web site
5. Go to the root of your web site, you should see json response of weather data. This data doesn't come from SQL.
6. Go to /api/todoitems, you should see an empty array `[]` if you just bootstrapped a clean database. If the web app can't access SQL DB, you will see error instead.

