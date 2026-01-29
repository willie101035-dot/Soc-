## PWM
''' vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity pingpong_top is
    port(
        clk          : in  std_logic; 
        rst          : in  std_logic; 
        btn_l        : in  std_logic; 
        btn_r        : in  std_logic; 
        led_out      : out std_logic_vector(7 downto 0) -- 兼用於顯示球與分數
    );
end pingpong_top;

architecture Behavioral of pingpong_top is
    signal div_reg  : unsigned(26 downto 0) := (others => '0');
    signal slow_clk : std_logic;
    
    signal ball_reg : std_logic_vector(7 downto 0) := "10000000";
    signal dir      : std_logic := '0'; 
    signal score_l  : unsigned(3 downto 0) := "0000";
    signal score_r  : unsigned(3 downto 0) := "0000";
    
    type state_type is (PLAYING, SHOW_SCORE);
    signal current_state : state_type := PLAYING;
    signal delay_cnt     : integer range 0 to 10 := 0; -- 控制分數顯示多久

begin
    process(clk)
    begin
        if rising_edge(clk) then
            div_reg <= div_reg + 1;
        end if;
    end process;
    slow_clk <= div_reg(24); 

    process(slow_clk, rst)
    begin
        if rst = '1' then
            ball_reg <= "10000000";
            score_l <= "0000";
            score_r <= "0000";
            current_state <= PLAYING;
            delay_cnt <= 0;
        elsif rising_edge(slow_clk) then
            case current_state is
                when PLAYING =>
                    if dir = '0' then 
                        if ball_reg = "00000010" then
                            if btn_r = '1' then 
                                dir <= '1'; ball_reg <= "00000100";
                            else ball_reg <= "00000001"; end if;
                        elsif ball_reg = "00000001" then 
                            score_l <= score_l + 1;
                            current_state <= SHOW_SCORE;
                        else
                            ball_reg <= '0' & ball_reg(7 downto 1);
                        end if;
                    else 
                        if ball_reg = "01000000" then
                            if btn_l = '1' then
                                dir <= '0'; ball_reg <= "00100000";
                            else ball_reg <= "10000000"; end if;
                        elsif ball_reg = "10000000" then 
                            score_r <= score_r + 1;
                            current_state <= SHOW_SCORE;
                        else
                            ball_reg <= ball_reg(6 downto 0) & '0';
                        end if;
                    end if;

                when SHOW_SCORE =>
                    if delay_cnt < 3 then
                        delay_cnt <= delay_cnt + 1;
                    else
                        delay_cnt <= 0;
                        ball_reg <= "10000000"; 
                        dir <= '0';
                        current_state <= PLAYING;
                    end if;
            end case;
        end if;
    end process;

    led_out <= ball_reg when current_state = PLAYING else 
               std_logic_vector(score_l) & std_logic_vector(score_r);

end Behavioral;
'''
