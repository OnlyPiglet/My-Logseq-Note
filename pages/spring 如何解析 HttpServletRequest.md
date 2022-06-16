- ```java
      FormHttpMessageConverter formHttpMessageConverter = new FormHttpMessageConverter();
  
      GsonHttpMessageConverter gsonHttpMessageConverter = new GsonHttpMessageConverter();
  
  
      private Map<String,Object>  getParams(HttpServletRequest request,Map<String,Object> params) throws IOException {
  
              Enumeration paramNames = request.getParameterNames();
              while (paramNames.hasMoreElements()) {
                  String paramName = (String) paramNames.nextElement();
                  String[] paramValues = request.getParameterValues(paramName);
                  if (paramValues.length == 1) {
                      String paramValue = paramValues[0];
                      if (paramValue.length() != 0) {
                          params.put(paramName, paramValue);
                      }
                  }
              }
          if (HttpMethod.POST.name().equals(request.getMethod()) || HttpMethod.PUT.name().equals( request.getMethod())){
  
              String contentType = request.getContentType();
              MediaType type= StringUtils.hasText(contentType)? MediaType.valueOf(contentType):null;
              ServletServerHttpRequest serverHttpRequest = new ServletServerHttpRequest(request);
  
              if (formHttpMessageConverter.canRead(MultiValueMap.class,type)) {
                  MultiValueMap<String, String> xmlParamMap = formHttpMessageConverter.read(null, serverHttpRequest);
                  params.putAll(xmlParamMap);
  
              }else if(gsonHttpMessageConverter.canRead(MultiValueMap.class,type)){
                  Map<String,Object> jsonParamsMaps = (Map<String, Object>) gsonHttpMessageConverter.read(Map.class,serverHttpRequest);
                  params.putAll(jsonParamsMaps);
              }
          }
              return params;
  
      }
  
  ```