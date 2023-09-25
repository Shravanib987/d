This is Response Validator

/*
 ****************************************************************************
 *
 * Copyright (c)2020 The Vanguard Group of Investment Companies (VGI)
 * All rights reserved.
 *
 * This source code is CONFIDENTIAL and PROPRIETARY to VGI. Unauthorized
 * distribution, adaptation, or use may be subject to civil and criminal
 * penalties.
 *
 ****************************************************************************
 Module Description:

 $HeadURL:$
 $LastChangedRevision:$
 $Author:$
 $LastChangedDate:$
*/
package com.vanguard.retail.legal.webservice.service.validator;

import javax.inject.Named;
import javax.inject.Singleton;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.Response.Status.Family;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.vanguard.retail.legal.webservice.exception.ProcessException;

@Named
@Singleton
public class ResponseValidator {

	private static final Logger logger = LoggerFactory.getLogger(ResponseValidator.class);

	public void validate(Response response) {

		if (response.getStatusInfo().getFamily() != Family.SUCCESSFUL) {

			logger.error("Unsuccessful service call.|status={}", response.getStatus());
			throw new ProcessException("Unsuccessful service call.|status=" + response.getStatus(), response.getStatus());
		}
	}
}





This is ChainedCreatableService.Java


/*
 ****************************************************************************
 *
 * Copyright (c)2021 The Vanguard Group of Investment Companies (VGI)
 * All rights reserved.
 *
 * This source code is CONFIDENTIAL and PROPRIETARY to VGI. Unauthorized
 * distribution, adaptation, or use may be subject to civil and criminal
 * penalties.
 *
 ****************************************************************************
 Module Description:

 $HeadURL:$
 $LastChangedRevision:$
 $Author:$
 $LastChangedDate:$
*/
package com.vanguard.retail.legal.webservice.service;

import static javax.ws.rs.client.Entity.json;
import static org.slf4j.LoggerFactory.getLogger;

import javax.inject.Inject;
import javax.inject.Named;
import javax.inject.Singleton;
import javax.servlet.http.HttpServletRequest;
import javax.ws.rs.core.Response;

import com.vanguard.retail.legal.webservice.exception.ProcessException;
import org.slf4j.Logger;

import com.vanguard.retail.legal.webservice.domain.Createable;
import com.vanguard.retail.legal.webservice.rest.ResponseFactory;
import com.vanguard.retail.legal.webservice.service.validator.ResponseValidator;
import com.vanguard.retail.legal.webservice.set.domain.EventTrackerDetails;

@Named
@Singleton
public class ChainedCreateableService {

	private final Logger log = getLogger(CreateableService.class);

	@Inject
	private ResponseFactory responseFactory;

	@Inject
	private ResponseValidator responseValidator;

	private String target;

	private String path;

	public ChainedCreateableService() {

	}

	public ChainedCreateableService(String target, String path) {
		this.target = target;
		this.path = path;
	}

	public Response create(Createable createable, HttpServletRequest httpServletRequest) {
		Response response = responseFactory.post(httpServletRequest, target, path, json(createable));
		validateResponse(response);
		EventTrackerDetails eventTrackerDetails = response.readEntity(EventTrackerDetails.class);
		log.info("Successfully called the {}{} to update the event details|eventId={}|status={}|type={}", target, path, createable.getEventId(), response.getStatus(), eventTrackerDetails.getType());
		return response;
	}


	private void validateResponse(Response response) {
		if (response != null) {
			EventTrackerDetails eventTrackerDetails = response.readEntity(EventTrackerDetails.class);
			if (eventTrackerDetails != null) {
				String message = eventTrackerDetails.getMessage();
				log.info("Successfully called the eventTracker to get the message {}", eventTrackerDetails.getMessage());

				if (eventTrackerDetails.getType() == null) {
					log.error("Error creating prospect:{}", message);
					ProcessException exception = new ProcessException(message, response.getStatus());
					exception.setErrorMessage(message);
					throw exception;
				}
			}
		}
		responseValidator.validate(response);
	}
}




This is ResponseValidatortest.java



/*
 ****************************************************************************
 *
 * Copyright (c)2020 The Vanguard Group of Investment Companies (VGI)
 * All rights reserved.
 *
 * This source code is CONFIDENTIAL and PROPRIETARY to VGI. Unauthorized
 * distribution, adaptation, or use may be subject to civil and criminal
 * penalties.
 *
 ****************************************************************************
 Module Description:

 $HeadURL:$
 $LastChangedRevision:$
 $Author:$
 $LastChangedDate:$
*/
package com.vanguard.retail.legal.webservice.service.validator;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

import javax.ws.rs.core.Response;
import javax.ws.rs.core.Response.Status;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.MockitoJUnitRunner;

import com.vanguard.retail.legal.webservice.exception.ProcessException;
@RunWith(MockitoJUnitRunner.class)
public class ResponseValidatorTest {

	@InjectMocks
	private ResponseValidator validator;
	
	@Mock
	private Response response;
	
	@Test
	public void testValidate(){
		when(response.getStatusInfo()).thenReturn(Status.OK);
		validator.validate(response);
		verify(response).getStatusInfo();
	}
	
	@Test(expected=ProcessException.class)
	public void testValidate_Exception(){
		when(response.getStatusInfo()).thenReturn(Status.INTERNAL_SERVER_ERROR);
		validator.validate(response);
	}
	
}



This is ChainedCreatableServicetest.java



/*
 ****************************************************************************
 *
 * Copyright (c)2021 The Vanguard Group of Investment Companies (VGI)
 * All rights reserved.
 *
 * This source code is CONFIDENTIAL and PROPRIETARY to VGI. Unauthorized
 * distribution, adaptation, or use may be subject to civil and criminal
 * penalties.
 *
 ****************************************************************************
 Module Description:

 $HeadURL:$
 $LastChangedRevision:$
 $Author:$
 $LastChangedDate:$
*/
package com.vanguard.retail.legal.webservice.service;

import static javax.ws.rs.client.Entity.json;
import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNotNull;
import static org.mockito.Mockito.doNothing;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;
import static org.powermock.reflect.Whitebox.getInternalState;
import static org.powermock.reflect.Whitebox.setInternalState;

import javax.servlet.http.HttpServletRequest;
import javax.ws.rs.core.Response;

import org.junit.Before;
import org.junit.Test;
import org.junit.experimental.categories.Category;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.mockito.junit.MockitoJUnitRunner;
import org.slf4j.Logger;

import com.vanguard.retail.legal.webservice.domain.Createable;
import com.vanguard.retail.legal.webservice.rest.ResponseFactory;
import com.vanguard.retail.legal.webservice.service.validator.ResponseValidator;
import com.vanguard.retail.legal.webservice.set.domain.EventTrackerDetails;
import com.vanguard.test.categories.UnitTest;

@Category(UnitTest.class)
@RunWith(MockitoJUnitRunner.class)
public class ChainedCreateableServiceTest {

	/*@InjectMocks*/ // does not work because of the additional constructor
	private ChainedCreateableService chainedChainedCreateableService;

	@Mock
	private Logger log;

	@Mock
	private ResponseFactory responseFactory;

	@Mock
	private ResponseValidator responseValidator;

	private String target = "target";

	private String path = "/path";

	@Before
	public void setUp() {
		chainedChainedCreateableService = new ChainedCreateableService(target, path);
		setInternalState(chainedChainedCreateableService, "log", log);
		setInternalState(chainedChainedCreateableService, "responseFactory", responseFactory);
		setInternalState(chainedChainedCreateableService, "responseValidator", responseValidator);
	}

	@Test
	public void givenAChainedCreateableServiceWhenCreatingANewInstanceThenTheNewInstanceIsNotNull() {
		chainedChainedCreateableService = new ChainedCreateableService();
		assertNotNull(chainedChainedCreateableService);
	}

	@Test
	public void givenATargetAndAPathWhenCreatingANewChainedCreateableServiceInstanceThenTheInstanceIsNotNullAndTheTargetAndPathAreSet() {
		assertNotNull(chainedChainedCreateableService);
		assertEquals(target, getInternalState(chainedChainedCreateableService, "target", ChainedCreateableService.class));
		assertEquals(path, getInternalState(chainedChainedCreateableService, "path", ChainedCreateableService.class));
	}

	@Test
	public void givenAChainedCreateableAndAHttpServletRequestWhenCreatingThenABaseResponseIsReturned() {
		String target = "target";
		String path = "/path";
		setInternalState(chainedChainedCreateableService, "target", target);
		setInternalState(chainedChainedCreateableService, "path", path);
		Createable createable = mock(Createable.class);
		HttpServletRequest httpServletRequest = mock(HttpServletRequest.class);
		String eventId = "eventId";
		EventTrackerDetails eventTrackerDetails = mock(EventTrackerDetails.class);
		int status = 200;
		String type = "type";
		String message = "message";
		Response expected = mock(Response.class);

		when(responseFactory.post(httpServletRequest, target, path, json(createable))).thenReturn(expected);
		doNothing().when(responseValidator).validate(expected);
		when(expected.readEntity(EventTrackerDetails.class)).thenReturn(eventTrackerDetails);
		when(createable.getEventId()).thenReturn(eventId);
		when(expected.getStatus()).thenReturn(status);
		when(eventTrackerDetails.getType()).thenReturn(type);
		when(eventTrackerDetails.getMessage()).thenReturn(message);
		doNothing().when(log).info("Successfully called the {}{} to update the event details|eventId={}|status={}|type={}|message={}", target, path, eventId, status, type, message);

		Response actual = chainedChainedCreateableService.create(createable, httpServletRequest);

		verify(responseFactory, times(1)).post(httpServletRequest, target, path, json(createable));
		verify(responseValidator, times(1)).validate(expected);
		verify(expected, times(1)).readEntity(EventTrackerDetails.class);
		verify(createable, times(1)).getEventId();
		verify(expected, times(1)).getStatus();
		verify(eventTrackerDetails, times(1)).getType();
		verify(eventTrackerDetails, times(1)).getMessage();
		verify(log, times(1)).info("Successfully called the {}{} to update the event details|eventId={}|status={}|type={}|message={}", target, path, eventId, status, type, message);

		assertEquals(expected, actual);
	}

}




Please correct the both test files and provide as per the ResponseValidator.java and ChainedCreatableService.java


