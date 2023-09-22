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
