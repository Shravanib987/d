import org.junit.Before;
import org.junit.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.slf4j.Logger;

import javax.ws.rs.core.Response;
import javax.ws.rs.core.Response.Status;

import static org.mockito.Mockito.*;

public class ResponseValidatorTest {

    @Mock
    private Logger logger;

    @Mock
    private Response response;

    private ResponseValidator responseValidator;

    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
        responseValidator = new ResponseValidator();
        responseValidator.setLogger(logger);
    }

    @Test
    public void testValidate_SuccessfulResponse() {
        when(response.getStatusInfo().getFamily()).thenReturn(Status.Family.SUCCESSFUL);

        // Call the validate method
        responseValidator.validate(response);

        // Verify that the logger was not called with an error message
        verify(logger, never()).error(anyString(), any());
    }

    @Test(expected = ProcessException.class)
    public void testValidate_UnsuccessfulResponse() {
        when(response.getStatusInfo().getFamily()).thenReturn(Status.Family.SERVER_ERROR);

        // Call the validate method, which should throw ProcessException
        responseValidator.validate(response);
    }
}
