    package com.stripe.sample;

    import java.nio.file.Paths;

    import static spark.Spark.get;
    import static spark.Spark.post;
    import static spark.Spark.staticFiles;
    import static spark.Spark.port;

    import com.google.gson.Gson;
    import com.google.gson.annotations.SerializedName;

    import com.stripe.Stripe;
    import com.stripe.model.PaymentIntent;
    import com.stripe.param.PaymentIntentCreateParams;
    import com.stripe.model.terminal.ConnectionToken;
    import com.stripe.param.terminal.ConnectionTokenCreateParams;
    import com.stripe.model.terminal.Location;
    import com.stripe.param.terminal.LocationCreateParams;
    import com.stripe.exception.StripeException;

    import java.util.Map;
    import java.util.HashMap;

    public class Server {
    private static Gson gson = new Gson();

    static class PaymentIntentParams {
    private String id;
    private long amount;
    public String getID() {
      return id;
    }

    public long getAmount() {
      return amount;
    }
 
    // This is your live secret API key.
    Stripe.apiKey = "pk_live_51KR9pNHUhMG4O3VamYiVYmh7qnY7Nf2x5oaENOZuuP18t0FVoinvUzB04nRO6ahvwGDDlWmgkvA3WgWnblSRKNb6003D3B5r0V";

    // The ConnectionToken's secret lets you connect to any Stripe Terminal reader
    // and take payments with your Stripe account.
    // Be sure to authenticate the endpoint for creating connection tokens.
    post("/connection_token", (request, response) -> {
      response.type("application/json");
      ConnectionTokenCreateParams params = ConnectionTokenCreateParams.builder()
        .build();

      ConnectionToken connectionToken = ConnectionToken.create(params);

      Map<String, String> map = new HashMap();
      map.put("secret", connectionToken.getSecret());

      return gson.toJson(map);
    });
    post("/create_payment_intent", (request, response) -> {
      response.type("application/json");

      PaymentIntentParams postBody = gson.fromJson(request.body(), PaymentIntentParams.class);

      // For Terminal payments, the 'payment_method_types' parameter must include
      // 'card_present' and the 'capture_method' must be set to 'manual'
      PaymentIntentCreateParams createParams = PaymentIntentCreateParams.builder()
        .setCurrency("aud")
        .setAmount(postBody.getAmount())
        .setCaptureMethod(PaymentIntentCreateParams.CaptureMethod.MANUAL)
        .addPaymentMethodType("card_present")
        .build();
      // Create a PaymentIntent with the order amount and currency
      PaymentIntent intent= PaymentIntent.create(createParams);

      Map<String, String> map = new HashMap();
      map.put("client_secret", intent.getClientSecret());

      return gson.toJson(map);
    });
    post("/capture_payment_intent", (request, response) -> {
      response.type("application/json");

      PaymentIntentParams postBody = gson.fromJson(request.body(), PaymentIntentParams.class);

      PaymentIntent intent = PaymentIntent.retrieve(postBody.getID());
      intent = intent.capture();

      return intent.toJson();
    });

    public static Location createLocation() throws StripeException{
    LocationCreateParams.Address address =
    LocationCreateParams.Address.builder()
      .setLine1("33 cabernet cres")
      .setCity("bundoora")
      .setState("Victoria")
      .setCountry("AU")
      .setPostalCode("3083")
      .build();

    LocationCreateParams params =
      LocationCreateParams.builder()
        .setDisplayName("HQ")
        .setAddress(address)
        .build();

    Location location = Location.create(params);

    return location;}
