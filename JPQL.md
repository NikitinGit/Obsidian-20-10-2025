1. Чтобы выбрать записи из events даже если у него events.country = null достаточно сделать запрос LEFT JOIN по id 
   @Query("SELECT new com.strikerstat.webapp.dto.open_events.EventPageDto(" +  
        "e.nameEvent, e.aboutEvent, e.address, e.performancePlace, e.performanceTime, " +  
        "e.eventDate, e.ringsCount, e.payBySite, e.version, " +  
        "s.sectionAgeName1, s.sectionAgeName2, s.sectionAgeName3, s.sectionAgeName4, " +  
        "c.name) " +  
        "FROM Event e " +  
        "INNER JOIN EventSetting s ON e.eventId = s.eventId " +  
        "LEFT JOIN Country c ON e.country.id = c.id " +  
  
        "WHERE e.eventId = :eventId")  
Optional<EventPageDto> getEventPageDto(@Param("eventId") Integer eventId);

2. Отличие LEFT JOIN fetch от LEFT JOIN - fetch избавляет от проблемы N + !
3. `LEFT OUTER JOIN FighterRating fr ON b.idBattle = fr.idBattle`- выбираются все записи из b но не все из FighterRating
4. JOIN == INNER JOIN ???
5. @Modifying @Query("DELETE FROM FighterRating f WHERE f.battleId = :battleId") работает быстрее чем List<FighterRating> fighterRatings = fighterRatingRepository.findRatesByBattleId(battleId);  
fighterRatingRepository.deleteAll(fighterRatings); ? и без каскадного удаления ?