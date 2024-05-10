# ct

import org.apache.http.HttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

public class ALMTestUpdater {

    private static final String ALM_URL = "https://your-alm-server/qcbin";
    private static final String DOMAIN = "your_domain";
    private static final String PROJECT = "your_project";
    private static final String USERNAME = "your_username";
    private static final String PASSWORD = "your_password";

    public static void main(String[] args) {
        try (CloseableHttpClient httpClient = HttpClients.createDefault()) {

            // Authenticate
            String auth = authenticate(httpClient);
            if (auth == null) {
                System.out.println("Authentication failed.");
                return;
            }

            // Update Test Result
            updateTestResult(httpClient, auth, "test_instance_id", "Passed");

            // Logout
            logout(httpClient);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static String authenticate(CloseableHttpClient httpClient) throws Exception {
        String authUrl = ALM_URL + "/authentication-point/authenticate";
        HttpGet authRequest = new HttpGet(authUrl);
        String auth = USERNAME + ":" + PASSWORD;
        String encodedAuth = new String(java.util.Base64.getEncoder().encode(auth.getBytes()));
        authRequest.setHeader("Authorization", "Basic " + encodedAuth);

        HttpResponse authResponse = httpClient.execute(authRequest);
        int statusCode = authResponse.getStatusLine().getStatusCode();
        EntityUtils.consume(authResponse.getEntity());

        if (statusCode == 200) {
            System.out.println("Authentication successful.");
            return encodedAuth;
        } else {
            System.out.println("Authentication failed with status code: " + statusCode);
            return null;
        }
    }

    private static void updateTestResult(CloseableHttpClient httpClient, String auth, String testInstanceId, String status) throws Exception {
        String updateUrl = ALM_URL + "/rest/domains/" + DOMAIN + "/projects/" + PROJECT + "/runs";
        HttpPost updateRequest = new HttpPost(updateUrl);
        updateRequest.setHeader("Authorization", "Basic " + auth);
        updateRequest.setHeader("Content-Type", "application/xml");

        String xmlPayload = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>"
            + "<Entity Type=\"run\">"
            + "<Fields>"
            + "<Field Name=\"name\"><Value>Test Run</Value></Field>"
            + "<Field Name=\"test-instance-id\"><Value>" + testInstanceId + "</Value></Field>"
            + "<Field Name=\"status\"><Value>" + status + "</Value></Field>"
            + "</Fields>"
            + "</Entity>";

        updateRequest.setEntity(new StringEntity(xmlPayload));

        HttpResponse updateResponse = httpClient.execute(updateRequest);
        int statusCode = updateResponse.getStatusLine().getStatusCode();
        EntityUtils.consume(updateResponse.getEntity());

        if (statusCode == 201) {
            System.out.println("Test result updated successfully.");
        } else {
            System.out.println("Failed to update test result with status code: " + statusCode);
        }
    }

    private static void logout(CloseableHttpClient httpClient) throws Exception {
        String logoutUrl = ALM_URL + "/authentication-point/logout";
        HttpGet logoutRequest = new HttpGet(logoutUrl);

        HttpResponse logoutResponse = httpClient.execute(logoutRequest);
        int statusCode = logoutResponse.getStatusLine().getStatusCode();
        EntityUtils.consume(logoutResponse.getEntity());

        if (statusCode == 200) {
            System.out.println("Logout successful.");
        } else {
            System.out.println("Logout failed with status code: " + statusCode);
        }
    }
}
