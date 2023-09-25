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
